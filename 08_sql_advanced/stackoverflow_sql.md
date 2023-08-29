# Сложные SQL запросы

**Проект состоит из SQL запросов к базе данных StackOverflow** — сервиса вопросов и ответов о программировании. StackOverflow похож на социальную сеть — пользователи сервиса задают вопросы, отвечают на посты, оставляют комментарии и ставят оценки другим ответам. Запросы направлены к той версии базы, в которой хранятся данные о постах за 2008 год, но в таблицах можно найти информацию и о более поздних оценках, которые эти посты получили.

ER-диаграмма и подробное описание к ней приложено в файле [stackoverflow_sql_description.pdf](https://yadi.sk/i/LGa68u6PcFGylQ)

1. Найдите количество вопросов, которые набрали больше 300 очков или как минимум 100 раз были добавлены в «Закладки».

```SELECT COUNT(id)
FROM stackoverflow.posts
WHERE post_type_id = 1 AND (score > 300 OR favorites_count >= 100);
```

2. Сколько в среднем в день задавали вопросов с 1 по 18 ноября 2008 года включительно? Результат округлите до целого числа.

```SELECT ROUND(AVG(posts_cnt))
FROM (
    SELECT creation_date::date AS dt, COUNT(id) AS posts_cnt
    FROM stackoverflow.posts
    WHERE post_type_id = 1 AND creation_date BETWEEN '2008-11-01' AND '2008-11-19'
    GROUP BY dt
    ORDER BY dt
) AS a;
```

3. Сколько пользователей получили значки сразу в день регистрации? Выведите количество уникальных пользователей.

```SELECT COUNT(DISTINCT sb.user_id)
FROM stackoverflow.badges AS sb
JOIN stackoverflow.users AS su ON sb.user_id = su.id
WHERE sb.creation_date::date = su.creation_date::date;
```

4. Сколько уникальных постов пользователя с именем Joel Coehoorn получили хотя бы один голос?

```WITH name AS (
    SELECT id
    FROM stackoverflow.users
    WHERE display_name = 'Joel Coehoorn'
)
SELECT COUNT(DISTINCT posts.id)
FROM (
    SELECT sp.id
    FROM stackoverflow.posts AS sp
    JOIN name ON sp.user_id = name.id
) AS posts
JOIN stackoverflow.votes AS sv ON posts.id = sv.post_id;
```

5. Выгрузите все поля таблицы vote_types. Добавьте к таблице поле rank, в которое войдут номера записей в обратном порядке. Таблица должна быть отсортирована по полю id.

```SELECT *, RANK() OVER (ORDER BY id DESC) AS rank
FROM stackoverflow.vote_types
ORDER BY id;
```

6. Отберите 10 пользователей, которые поставили больше всего голосов типа Close. Отобразите таблицу из двух полей: идентификатор пользователя и количество голосов. Отсортируйте данные сначала по убыванию количества голосов, потом по убыванию значения идентификатора пользователя.

```SELECT user_id, COUNT(id) AS votes
FROM stackoverflow.votes
WHERE vote_type_id = 6
GROUP BY user_id
ORDER BY votes DESC, user_id DESC
LIMIT 10;
```

7. Отберите 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно. Отобразите несколько полей:
- идентификатор пользователя;
- число значков;
- место в рейтинге — чем больше значков, тем выше рейтинг.
Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге.

```SELECT *, DENSE_RANK() OVER (ORDER BY badges_cnt DESC) AS rank
FROM (
    SELECT user_id, COUNT(id) AS badges_cnt
    FROM stackoverflow.badges
    WHERE creation_date::date BETWEEN '2008-11-15' AND '2008-12-15'
    GROUP BY user_id
    ORDER BY badges_cnt DESC
) AS a
LIMIT 10;
```

8. Сколько в среднем очков получает пост каждого пользователя? Сформируйте таблицу из следующих полей:
- заголовок поста;
- идентификатор пользователя;
- число очков поста;
- среднее число очков пользователя за пост, округлённое до целого числа.
Не учитывайте посты без заголовка, а также те, что набрали ноль очков.

```SELECT p.title, p.user_id, p.score,
       ROUND(AVG(p.score) OVER (PARTITION BY p.user_id))
FROM stackoverflow.posts AS p
WHERE p.title != '' AND p.score != 0;
```

9. Отобразите заголовки постов, которые были написаны пользователями, получившими более 1000 значков. Посты без заголовков не должны попасть в список.

```WITH popular AS (
    SELECT user_id
    FROM (
        SELECT DISTINCT user_id, COUNT(id) AS badges
        FROM stackoverflow.badges
        GROUP BY user_id
    ) AS a
    WHERE badges > 1000
)
SELECT title
FROM stackoverflow.posts AS sp
JOIN popular ON sp.user_id = popular.user_id
WHERE title != '';
```

10. Напишите запрос, который выгрузит данные о пользователях из США (англ. United States). Разделите пользователей на три группы в зависимости от количества просмотров их профилей:
- пользователям с числом просмотров больше либо равным 350 присвойте группу 1;
- пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;
- пользователям с числом просмотров меньше 100 — группу 3.
Отобразите в итоговой таблице идентификатор пользователя, количество просмотров профиля и группу. Пользователи с нулевым количеством просмотров не должны войти в итоговую таблицу.

```SELECT id, views, 
       CASE
           WHEN views >= 350 THEN 1
           WHEN views >= 100 THEN 2
           ELSE 3
       END AS view_group
FROM stackoverflow.users AS u
WHERE location LIKE '%united states%' AND views > 0;
```

11. Дополните предыдущий запрос. Отобразите лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе. Выведите поля с идентификатором пользователя, группой и количеством просмотров. Отсортируйте таблицу по убыванию просмотров, а затем по возрастанию значения идентификатора.
    
```WITH s AS (
    SELECT id,
           views,
           CASE
               WHEN views >= 350 THEN 1
               WHEN views >= 100 THEN 2
               ELSE 3
           END AS view_group,
           MAX(views) OVER (PARTITION BY
               CASE
                   WHEN views >= 350 THEN 1
                   WHEN views >= 100 THEN 2
                   ELSE 3
               END) AS max_group
    FROM stackoverflow.users AS u
    WHERE location LIKE '%united states%' AND views > 0
    ORDER BY views DESC, CASE
                                WHEN views >= 350 THEN 1
                                WHEN views >= 100 THEN 2
                                ELSE 3
                            END ASC
)
SELECT id,
       views,
       view_group
FROM s
WHERE views = max_group
GROUP BY 1, 2, 3
ORDER BY views DESC, view_group ASC;
```

12. Посчитайте ежедневный прирост новых пользователей в ноябре 2008 года. Сформируйте таблицу с полями:
- номер дня;
- число пользователей, зарегистрированных в этот день;
- сумму пользователей с накоплением.

```WITH s AS (
    SELECT EXTRACT(day FROM creation_date) AS dt,
           id,
           COUNT(id) AS count,
           SUM(COUNT(id)) OVER (ORDER BY EXTRACT(isodow FROM creation_date)) AS sum_users
    FROM stackoverflow.users
    WHERE creation_date BETWEEN '2008-11-01' AND '2008-12-01'
    GROUP BY 1
)
SELECT dt,
       count,
       sum_users
FROM s;
```

13. Для каждого пользователя, который написал хотя бы один пост, найдите интервал между регистрацией и временем создания первого поста. Отобразите:
идентификатор пользователя;
разницу во времени между регистрацией и первым постом.

```WITH sec AS (
    SELECT u.id,
           MIN(u.creation_date) AS dt_reg,
           MIN(p.creation_date) AS dt_pos
    FROM stackoverflow.users AS u
    INNER JOIN stackoverflow.posts AS p ON u.id = p.user_id
    GROUP BY u.id
)
SELECT id,
       dt_pos - dt_reg
FROM sec;
```

14. Выведите общую сумму просмотров постов за каждый месяц 2008 года. Если данных за какой-либо месяц в базе нет, такой месяц можно пропустить. Результат отсортируйте по убыванию общего количества просмотров.

```SELECT DISTINCT
    CAST(date_trunc('month', creation_date) AS DATE) AS month,
    SUM(views_count) AS views
FROM stackoverflow.posts
WHERE EXTRACT(year FROM creation_date) = 2008
GROUP BY month
ORDER BY views DESC;
```

15. Выведите имена самых активных пользователей, которые в первый месяц после регистрации (включая день регистрации) дали больше 100 ответов. Вопросы, заданные пользователями, не учитывайте. Для каждого имени пользователя выведите количество уникальных значений user_id. Отсортируйте результат по полю с именами в лексикографическом порядке.

```SELECT
    u.display_name,
    COUNT(DISTINCT p.user_id)
FROM stackoverflow.posts p
JOIN stackoverflow.post_types pt ON p.post_type_id = pt.id
JOIN stackoverflow.users u ON u.id = p.user_id
WHERE date_trunc('day', p.creation_date) >= date_trunc('day', u.creation_date)
  AND date_trunc('day', p.creation_date) <= date_trunc('day', u.creation_date) + INTERVAL '1 month'
  AND pt.type = 'answer'
GROUP BY u.display_name
HAVING COUNT(*) > 100
ORDER BY u.display_name;
```

16. Выведите количество постов за 2008 год по месяцам. Отберите посты от пользователей, зарегистрировавшихся в сентябре 2008 года и сделавших хотя бы один пост в декабре того же года. Отсортируйте таблицу по значению месяца по убыванию.

```SELECT
    DATE_TRUNC('month', creation_date)::DATE AS month,
    COUNT(*)
FROM stackoverflow.posts
WHERE user_id IN (
    SELECT DISTINCT u.id
    FROM stackoverflow.users u
    JOIN stackoverflow.posts p ON u.id = p.user_id
    WHERE DATE_TRUNC('month', u.creation_date) = '2008-09-01'
      AND DATE_TRUNC('month', p.creation_date) = '2008-12-01'
)
GROUP BY month
ORDER BY month DESC;
```

17. Используя данные о постах, выведите следующие поля:
- идентификатор пользователя, который написал пост;
- дата создания поста;
- количество просмотров у текущего поста;
- сумму просмотров постов автора с накоплением.
Данные в таблице должны быть отсортированы по возрастанию идентификаторов пользователей, а данные об одном и том же пользователе — по возрастанию даты создания поста.

```WITH se AS (
    SELECT
        user_id,
        CAST(date_trunc('day', creation_date) AS DATE) AS dt,
        COUNT(CAST(date_trunc('day', creation_date) AS DATE))
    FROM stackoverflow.posts
    WHERE creation_date::DATE BETWEEN '2008-12-01' AND '2008-12-07'
    GROUP BY 1, 2
)
SELECT ROUND(AVG(count))
FROM se;
```

18. Насколько процентов менялось количество постов ежемесячно с 1 сентября по 31 декабря 2008 года? Отобразите таблицу со следующими полями:
- номер месяца;
- количество постов за месяц;
- процент, который показывает, насколько изменилось количество постов в текущем месяце по сравнению с предыдущим.
Если постов стало меньше, значение процента должно быть отрицательным, если больше — положительным. Округлите значение процента до двух знаков после запятой.
Напомним, что при делении одного целого числа на другое в PostgreSQL в результате получится целое число, округлённое до ближайшего целого вниз. Чтобы этого избежать, переведите делимое в тип numeric.

```WITH sec AS (
    SELECT
        EXTRACT(month FROM creation_date) AS dt,
        COUNT(views_count)
    FROM stackoverflow.posts
    WHERE CAST(date_trunc('month', creation_date) AS DATE) BETWEEN '2008-09-01' AND '2008-12-01'
    GROUP BY 1
)
SELECT *,
       ROUND(((count::NUMERIC / LAG(count) OVER (ORDER BY dt) - 1)) * 100, 2)
FROM sec;
```

19. Выгрузите данные об активности пользователя, который опубликовал больше всего постов за всё время. Выведите данные за октябрь 2008 года в следующем виде:
- номер недели;
- дата и время последнего поста, опубликованного на этой неделе.

```WITH a AS (
    SELECT p.user_id,
           COUNT(p.id) AS cnt
    FROM stackoverflow.posts AS p
    GROUP BY p.user_id
    ORDER BY cnt DESC LIMIT 1
)
SELECT DISTINCT
    EXTRACT(week FROM p.creation_date) AS week,
    MAX(p.creation_date) OVER (ORDER BY EXTRACT(week FROM p.creation_date))
FROM a
JOIN stackoverflow.posts AS p ON p.user_id = a.user_id
WHERE EXTRACT(month FROM p.creation_date) = 10
AND EXTRACT(year FROM p.creation_date) = 2008;
```

