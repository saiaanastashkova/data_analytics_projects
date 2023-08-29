# Базовые SQL запросы

[ER диаграмма БД]()

1. Посчитайте, сколько компаний закрылось.
SELECT COUNT(id)

FROM company

WHERE status = 'closed';

2. Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы company. Отсортируйте таблицу по убыванию значений в поле funding_total .
SELECT SUM(price_amount)
FROM acquisition
WHERE EXTRACT(YEAR FROM acquired_at) BETWEEN 2011 AND 2013 AND term_code = 'cash';

3. Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.
SELECT SUM(price_amount)
FROM acquisition
WHERE EXTRACT(YEAR FROM acquired_at) BETWEEN 2011 AND 2013 AND term_code = 'cash';

4. Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на 'Silver'.
SELECT first_name, last_name, twitter_username
FROM people
WHERE twitter_username LIKE 'Silver%';

5. Выведите на экран всю информацию о людях, у которых названия аккаунтов в твиттере содержат подстроку 'money', а фамилия начинается на 'K'.
SELECT *
FROM people
WHERE twitter_username LIKE '%money%' AND last_name LIKE 'K%';

6. Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.
SELECT country_code, SUM(funding_total) AS funding_total
FROM company
GROUP BY country_code
ORDER BY funding_total DESC;

7. Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.
SELECT funded_at, MIN(raised_amount) AS min, MAX(raised_amount) AS max
FROM funding_round
GROUP BY funded_at
HAVING min(raised_amount) != 0 AND min(raised_amount) != max(raised_amount);

8. Создайте поле с категориями:
Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.
Отобразите все поля таблицы fund и новое поле с категориями.
SELECT *,
    CASE
        WHEN invested_companies >= 100 THEN 'high_activity'
        WHEN invested_companies BETWEEN 20 AND 99 THEN 'middle_activity'
        WHEN invested_companies < 20 THEN 'low_activity'
    END AS activity
FROM fund;

9. Для каждой из категорий, назначенных в предыдущем задании, посчитайте округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие. Выведите на экран категории и среднее число инвестиционных раундов. Отсортируйте таблицу по возрастанию среднего.
SELECT activity, ROUND(avg_round)
FROM (
    SELECT AVG(investment_rounds) AS avg_round,
        CASE
            WHEN invested_companies >= 100 THEN 'high_activity'
            WHEN invested_companies >= 20 THEN 'middle_activity'
            ELSE 'low_activity'
        END AS activity
    FROM fund
    GROUP BY activity
    ORDER BY avg_round
) AS a;

10. Проанализируйте, в каких странах находятся фонды, которые чаще всего инвестируют в стартапы.
Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды этой страны, основанные с 2010 по 2012 год включительно. Исключите страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. Выгрузите десять самых активных стран-инвесторов.
Отсортируйте таблицу по среднему количеству компаний от большего к меньшему, а затем по коду страны в лексикографическом порядке.
SELECT *
FROM (
    SELECT country_code, MIN(invested_companies) AS min,
        MAX(invested_companies) AS max, AVG(invested_companies) AS avg
    FROM fund
    WHERE EXTRACT(YEAR FROM founded_at) BETWEEN 2010 AND 2012
    GROUP BY country_code
    HAVING min(invested_companies) != 0
    ORDER BY avg DESC, country_code
    LIMIT 10
) AS top_countries;

11. Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.
SELECT p.first_name,
    p.last_name,
    e.instituition
FROM people AS p
LEFT JOIN education AS e ON p.id = e.person_id;

12. Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.
WITH a AS (
    SELECT id, company_id FROM people
), b AS (
    SELECT person_id, instituition FROM education
), c AS (
    SELECT name, id FROM company
)
SELECT c.name, d.instituitions
FROM c
INNER JOIN (
    SELECT a.company_id, COUNT(DISTINCT b.instituition) AS instituitions
    FROM a
    INNER JOIN b ON b.person_id = a.id
    GROUP BY company_id
) AS d ON c.id = d.company_id
ORDER BY d.instituitions DESC
LIMIT 5;

13. Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.
WITH a AS (
    SELECT name, id
    FROM company
    WHERE status='closed'
),
b AS (
    SELECT company_id
    FROM funding_round
    WHERE is_first_round = is_last_round
      AND is_first_round = 1
      AND is_last_round = 1
)
SELECT DISTINCT(name)
FROM a
INNER JOIN b ON a.id = b.company_id;

14. Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.
WITH a AS (
    SELECT name, id
    FROM company
    WHERE status='closed'
),
b AS (
    SELECT company_id
    FROM funding_round
    WHERE is_first_round = 1
      AND is_last_round = 1
),
c AS (
    SELECT id
    FROM people
    WHERE company_id IN (
        SELECT id
        FROM a
        INNER JOIN b ON a.id = b.company_id
    )
)
SELECT DISTINCT c.id, d.instituition
FROM c
INNER JOIN education AS d ON c.id = d.person_id;

15. Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.
WITH a AS (
    SELECT name, id
    FROM company
    WHERE status='closed'
),
b AS (
    SELECT company_id
    FROM funding_round
    WHERE is_first_round = 1
      AND is_last_round = 1
),
c AS (
    SELECT id
    FROM people
    WHERE company_id IN (
        SELECT id
        FROM a
        INNER JOIN b ON a.id = b.company_id
    )
)
SELECT DISTINCT c.id, d.instituition
FROM c
INNER JOIN education AS d ON c.id = d.person_id;

16. Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания. При подсчёте учитывайте, что некоторые сотрудники могли окончить одно и то же заведение дважды.
WITH a AS (
    SELECT name, id
    FROM company
    WHERE status='closed'
),
b AS (
    SELECT company_id
    FROM funding_round
    WHERE is_first_round = 1
      AND is_last_round = 1
),
c AS (
    SELECT id
    FROM people
    WHERE company_id IN (
        SELECT id
        FROM a
        INNER JOIN b ON a.id = b.company_id
    )
)
SELECT id, COUNT(instituition)
FROM (
    SELECT c.id, d.instituition
    FROM c
    INNER JOIN education AS d ON c.id = d.person_id
) AS e
GROUP BY id;

17. Дополните предыдущий запрос и выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.
WITH a AS (
    SELECT name, id
    FROM company
    WHERE status='closed'
),
b AS (
    SELECT company_id
    FROM funding_round
    WHERE is_first_round = 1
      AND is_last_round = 1
)
SELECT AVG(instituitions)
FROM (
    SELECT id, COUNT(instituition) AS instituitions
    FROM (
        SELECT c.id, d.instituition
        FROM (
            SELECT id
            FROM people
            WHERE company_id IN (
                SELECT id
                FROM a
                INNER JOIN b ON a.id = b.company_id
            )
        ) AS c
        INNER JOIN education AS d ON c.id = d.person_id
    ) AS e
    GROUP BY id
) AS f;

18. Напишите похожий запрос: выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники Facebook.
SELECT f.name AS name_of_fund, c.name AS name_of_company, fr.raised_amount AS amount
FROM investment AS i
LEFT JOIN company AS c ON c.id = i.company_id
LEFT JOIN fund AS f ON i.fund_id = f.id
INNER JOIN (
    SELECT *
    FROM funding_round
    WHERE funded_at BETWEEN '2012-01-01' AND '2013-12-31'
) AS fr ON fr.id = i.funding_round_id
WHERE c.milestones > 6;

WITH a AS (
    SELECT id AS facebook_id
    FROM company
    WHERE name = 'facebook'
),
b AS (
    SELECT person_id, instituition
    FROM education
)
SELECT AVG(instituitions)
FROM (
    SELECT person_id, COUNT(instituition) AS instituitions
    FROM b
    INNER JOIN (
        SELECT id
        FROM people AS p
        INNER JOIN a ON p.company_id = a.facebook_id
    ) AS f ON b.person_id = f.id
    GROUP BY person_id
) AS g;

19. Составьте таблицу из полей:
name_of_fund — название фонда;
name_of_company — название компании;
amount — сумма инвестиций, которую привлекла компания в раунде. В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.
SELECT f.name AS name_of_fund, c.name AS name_of_company, fr.raised_amount AS amount
FROM investment AS i
LEFT JOIN company AS c ON c.id = i.company_id
LEFT JOIN fund AS f ON i.fund_id = f.id
INNER JOIN (
    SELECT *
    FROM funding_round
    WHERE funded_at BETWEEN '2012-01-01' AND '2013-12-31'
) AS fr ON fr.id = i.funding_round_id
WHERE c.milestones > 6;

20. Выгрузите таблицу, в которой будут такие поля:
- название компании-покупателя;
- сумма сделки;
- название компании, которую купили;
- сумма инвестиций, вложенных в купленную компанию;
- доля, которая отображает, во сколько раз сумма покупки
превысила сумму вложенных в компанию инвестиций,
округлённая до ближайшего целого числа.
Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы.
Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в лексикографическом порядке. Ограничьте таблицу первыми десятью записями.
WITH acquiring AS (
    SELECT c.name AS buyer, a.price_amount AS price, a.id AS key
    FROM acquisition AS a
    LEFT JOIN company AS c ON a.acquiring_company_id = c.id
    WHERE a.price_amount > 0
),
acquired AS (
    SELECT c.name AS acquisition, c.funding_total AS investment, a.id AS key
    FROM acquisition AS a
    LEFT JOIN company AS c ON a.acquired_company_id = c.id
    WHERE c.funding_total > 0
)
SELECT acqn.buyer, acqn.price, acqd.acquisition, acqd.investment, ROUND(acqn.price / acqd.investment) AS uplift
FROM acquiring AS acqn
JOIN acquired AS acqd ON acqn.key = acqd.key
WHERE acqn.price > 0
ORDER BY price DESC, acquisition
LIMIT 10;

21. Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
- номер месяца, в котором проходили раунды;
- количество уникальных названий фондов из США, которые
инвестировали в этом месяце;
- количество компаний, купленных за этот месяц;
- общая сумма сделок по покупкам в этом месяце.
SELECT EXTRACT(MONTH FROM fr.funded_at) AS funding_month,
       COUNT(DISTINCT f.id) AS num_funds_usa,
       COUNT(DISTINCT acq.id) AS num_acquired_companies,
       SUM(acq.price_amount) AS total_acquisition_amount
FROM funding_round AS fr
LEFT JOIN company AS c ON fr.company_id = c.id
LEFT JOIN fund AS f ON fr.fund_id = f.id
LEFT JOIN acquisition AS acq ON c.id = acq.acquiring_company_id AND acq.price_amount <> 0
WHERE fr.funded_at BETWEEN '2010-01-01' AND '2013-12-31'
GROUP BY funding_month
ORDER BY funding_month;

22. Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
- номер месяца, в котором проходили раунды;
- количество уникальных названий фондов из США, которые
инвестировали в этом месяце;
- количество компаний, купленных за этот месяц;
- общая сумма сделок по покупкам в этом месяце.
SELECT EXTRACT(MONTH FROM fr.funded_at) AS funding_month,
       COUNT(DISTINCT f.id) AS num_funds_usa,
       COUNT(DISTINCT acq.id) AS num_acquired_companies,
       SUM(acq.price_amount) AS total_acquisition_amount
FROM funding_round AS fr
LEFT JOIN company AS c ON fr.company_id = c.id
LEFT JOIN fund AS f ON fr.fund_id = f.id
LEFT JOIN acquisition AS acq ON c.id = acq.acquiring_company_id AND acq.price_amount <> 0
WHERE fr.funded_at BETWEEN '2010-01-01' AND '2013-12-31'
GROUP BY funding_month
ORDER BY funding_month;

23. Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.
WITH yearly_avg AS (
    SELECT country_code,
           AVG(CASE WHEN EXTRACT(YEAR FROM founded_at) = 2011 THEN funding_total END) AS y_2011,
           AVG(CASE WHEN EXTRACT(YEAR FROM founded_at) = 2012 THEN funding_total END) AS y_2012,
           AVG(CASE WHEN EXTRACT(YEAR FROM founded_at) = 2013 THEN funding_total END) AS y_2013
    FROM company
    WHERE EXTRACT(YEAR FROM founded_at) IN (2011, 2012, 2013)
    GROUP BY country_code
)
SELECT y.country_code, y.y_2011, y.y_2012, y.y_2013
FROM yearly_avg AS y
ORDER BY y.y_2011 DESC;
