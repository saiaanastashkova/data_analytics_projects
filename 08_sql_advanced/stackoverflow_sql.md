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

6. Отберите 10 пользователей, которые поставили больше всего голосов типа Close. Отобразите таблицу из двух полей: идентификатор пользователя и количество голосов. Отсортируйте данные сначала по убыванию количества голосов, потом по убыванию значения идентификатора пользователя.
SELECT user_id, COUNT(id) AS votes
FROM stackoverflow.votes
WHERE vote_type_id = 6
GROUP BY user_id
ORDER BY votes DESC, user_id DESC
LIMIT 10;

7. Отберите 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно. Отобразите несколько полей:
- идентификатор пользователя;
- число значков;
- место в рейтинге — чем больше значков, тем выше рейтинг.
Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге.
SELECT *, DENSE_RANK() OVER (ORDER BY badges_cnt DESC) AS rank
FROM (
    SELECT user_id, COUNT(id) AS badges_cnt
    FROM stackoverflow.badges
    WHERE creation_date::date BETWEEN '2008-11-15' AND '2008-12-15'
    GROUP BY user_id
    ORDER BY badges_cnt DESC
) AS a
LIMIT 10;

8. Сколько в среднем очков получает пост каждого пользователя? Сформируйте таблицу из следующих полей:
- заголовок поста;
- идентификатор пользователя;
- число очков поста;
- среднее число очков пользователя за пост, округлённое до целого числа.
Не учитывайте посты без заголовка, а также те, что набрали ноль очков.
SELECT p.title, p.user_id, p.score,
       ROUND(AVG(p.score) OVER (PARTITION BY p.user_id))
FROM stackoverflow.posts AS p
WHERE p.title != '' AND p.score != 0;

9. Отобразите заголовки постов, которые были написаны пользователями, получившими более 1000 значков. Посты без заголовков не должны попасть в список.
WITH popular AS (
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

10. Напишите запрос, который выгрузит данные о пользователях из США (англ. United States). Разделите пользователей на три группы в зависимости от количества просмотров их профилей:
- пользователям с числом просмотров больше либо равным 350 присвойте группу 1;
- пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;
- пользователям с числом просмотров меньше 100 — группу 3.
Отобразите в итоговой таблице идентификатор пользователя, количество просмотров профиля и группу. Пользователи с нулевым количеством просмотров не должны войти в итоговую таблицу.
SELECT id, views, 
       CASE
           WHEN views >= 350 THEN 1
           WHEN views >= 100 THEN 2
           ELSE 3
       END AS view_group
FROM stackoverflow.users AS u
WHERE location LIKE '%united states%' AND views > 0;

11. Дополните предыдущий запрос. Отобразите лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе. Выведите поля с идентификатором пользователя, группой и количеством просмотров.
