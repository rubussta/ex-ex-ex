## stratascratch.com (PostgreSQL)
<details>
<summary>Упражнение "Share of Active Users": расчет доли с дополнительным условием</summary>
ID 2005<br>
<p>Output share of US users that are active. Active users are the ones with an "open" status in the table.</p>
<p>Выведите долю активных пользователей в США. Активные пользователи - это те, у кого в таблице статус "open".</p>
Table: fb_active_users<br>
user_id: int<br>
name: varchar<br>
status: varchar<br>
country: varchar

 **Solution**
```sql
-- Чтобы формула "заработала" нужно поривести одно из значений к numeric.
-- Вариант 1.

-- Использование условного выражения
SELECT
round(count(CASE WHEN status = 'open' THEN 1 ELSE NULL END) / count(user_id)::numeric, 1)  AS active_users_share
FROM fb_active_users;
```

```sql
-- Вариант 2.
-- Использование агрегатного выражения

SELECT
round(count(status) FILTER (WHERE status = 'open' ) / count(user_id)::numeric, 1) AS active_users_share
FROM fb_active_users;

```

 **Output**
 
|active_users_share|
|---|
|0.5|

</details>
<details>
<summary>Упражнение "New Products": сравнение периодов, выделенных из одной колонки таблицы</summary>
ID 10318<br>
<p>You are given a table of product launches by company by year. Write a query to count the net difference between the number of products companies launched in 2020 with the number of products companies launched in the previous year. Output the name of the companies and a net difference of net products released for 2020 compared to the previous year.</p>
<p>Вам предоставляется таблица запусков продуктов компаниями по годам. Напишите запрос, чтобы подсчитать чистую разницу между количеством продуктов, запущенных компаниями в 2020 году, и количеством продуктов, запущенных компаниями в предыдущем году. Выведите названия компаний и разницу в количестве продуктов, выпущенных за 2020 год, по сравнению с предыдущим годом.</p>

Table: car_launches</br>
year: int</br>
company_name: varchar</br>
product_name: varchar

 **Solution**
```sql
-- Вариант 1

WITH prod_2020 AS  --  products launched in 2020
(
SELECT  company_name, count(product_name) AS launched_2020
    FROM car_launches
    WHERE year = 2020
    GROUP BY company_name
),
prod_2019 AS --  products launched in 2019
(
SELECT  company_name, count(product_name) AS launched_2019
    FROM car_launches
    WHERE year = 2020 - 1
    GROUP BY company_name
)
SELECT  prod_2020.company_name, launched_2020 - launched_2019 AS net_products
    FROM prod_2020
    JOIN prod_2019 ON prod_2020.company_name = prod_2019.company_name
    ORDER BY prod_2020.company_name;

```

```sql
-- Вариант 2

SELECT prod_2020.company_name, count(DISTINCT prod_2020.product_name) - count(DISTINCT prod_2019.product_name) AS net_products
    FROM 
       (SELECT  company_name, product_name, year
        FROM car_launches
        WHERE year = 2020) AS prod_2020
    JOIN 
       ( SELECT  company_name, product_name, year
        FROM car_launches
        WHERE year = 2020 - 1) AS prod_2019
    ON prod_2020.company_name = prod_2019. company_name
    GROUP BY prod_2020.company_name
    ORDER BY company_name;

```
 **Output**
 
|company_name|net_products|
|---|---:|
|Chevrolet|2|
|Ford|-1|
Honda|-3|
|Jeep|1|
|Toyota|-1|

</details>
<details>
<summary>Упражнение "Premium Acounts": добавление к таблице колонок на основе данных этой же таблицы</summary>
ID 2097<br>
<p>You are given a dataset that provides the number of active users per day per premium account. 
A premium account will have an entry for every day that it’s premium. However, a premium account may be temporarily discounted and considered not paid, this is indicated by a value of 0 in the final_price column for a certain day. 
Find out how many premium accounts that are paid on any given day are still premium and paid 7 days later.</p>

<p>Output the date, the number of premium and paid accounts on that day, and the number of how many of these accounts are still premium and paid 7 days later. 
Since you are only given data for a 14 days period, only include the first 7 available dates in your output.</p>

Table: premium_accounts_by_day<br>
account_id: varchar<br>
entry_date: datetime<br>
users_visited_7d: int<br>
final_price: int<br>
plan_size: int

 **Solution**
```sql
-- Чтобы получить для решения две колонки, таблица объединяется сама с собой.
-- Далее отбираются нужные строки, которые агрегируются по дате.

SELECT 
	a.entry_date, 
	count(a.account_id) AS premium_paid_accounts,
	count(b.account_id) AS premium_paid_accounts_after_7d
FROM premium_accounts_by_day AS a
LEFT JOIN premium_accounts_by_day AS b ON a.account_id = b.account_id
AND b.final_price > 0
AND date_part('day', b.entry_date::timestamp - a.entry_date::timestamp) = 7
WHERE a.final_price > 0
GROUP BY a.entry_date
ORDER BY a.entry_date
LIMIT 7;

```
 **Output**
 
entry_date|premium_paid_accounts|premium_paid_accounts_after_7d|
|---|---:|---:|
2022-02-07|2|2|
2022-02-08|3|2|
2022-02-09|3|2|
2022-02-10|4|3|
2022-02-11|4|1|
2022-02-12|3|2|
2022-02-13|3|1|

</details>
<details>
<summary>Упражнение "Flags per Video": преобразование строковых данных с условным выражением и очисткой от возможных пробелов"</summary>
ID 2102<br>
<p>For each video, find how many unique users flagged it. A unique user can be identified using the combination of their first name and last name. Do not consider rows in which there is no flag ID.</p>
<p>Для каждого видео найдите, сколько уникальных пользователей отметили его. Уникального пользователя можно идентифицировать, используя комбинацию его имени и фамилии. Не рассматривайте строки, в которых нет идентификатора флага.</p>

Table: user_flags<br>
user_firstname: varchar<br>
user_lastname: varchar<br>
video_id: varchar<br>
flag_id: varchar

 **Solution**
```sql
-- Функция concat объединяет строки для формирования id. 
-- Условное выцражение COALESCE возвращает пустую строку, если нет имени или фамилии.
-- Функция trim удаляет пробелы слева и справа.

SELECT 
	video_id,
	count(DISTINCT concat(COALESCE(trim(FROM user_firstname), ''), COALESCE(trim(FROM user_lastname), ''))) num_unique_users
FROM user_flags
WHERE flag_id IS NOT NULL -- Отфильтровываем пользователей без флага
GROUP BY video_id;
```

 **Output**
 
|video_id|num_unique_users|
|:---|---:|
|5qap5aO4i9A|2|
|Ct6BUPvE2sM|2|
|dQw4w9WgXcQ|5|
|jNQXAC9IVRw|3|
|y6120QOlsfU|5|

</details>
<details>
<summary>Упражнение "User with Most Approved Flags": оконная функция dense_rank без PARTITION BY</summary>
ID 2104<br>
<p>Which user flagged the most distinct videos that ended up approved by YouTube? Output, in one column, their full name or names in case of a tie. In the user's full name, include a space between the first and the last name.</p>
<p>Какой пользователь отметил наиболее отличающиеся друг от друга видеоролики, которые в итоге были одобрены YouTube? Выведите в одном столбце их полное имя или фамилии в случае совпадения. В полном имени пользователя укажите пробел между именем и фамилией.</p>
<p>Table: user_flags
user_firstname: varchar<br>
user_lastname: varchar<br>
video_id: varchar<br>
flag_id: varchar</p>
<p>Table: flag_review<br>
flag_id: varchar<br>
reviewed_by_yt: bool<br>
reviewed_date: datetime<br>
reviewed_outcome: varchar</p>

 **Solution**
```sql
/*
По условию задания фактически нужно получить пользователя (или пользователей), у которого было больше всех уникальных видео, 
которые были одобрены к публикации. Таблица flag_review отражает результат одоброения.
Очевидно, что будет неизвестное кол-во пользователей с одинаковым числом роликов и нам неизвестен аргумент для LIMIT в связке с ORDER BY.
Поэтому, пользователей нужно проранжировать и получить список тех, у кого наивысший ранг.
Также вызывает вопрос идентификация пользователей по имени и фамилии по условию задачи. Но ID пользователя нет и используем, что есть. 

Объединяем таблицы и отфильтровыввем строки с пользователями, у которых видео одобрены.
Для ранжирования пользоватей используем оконную функцию dense_rank без PARTITION BY.
Без PARTITION BY строки раздела состоят из всех строк таблицы, которые группируются с помощью GROUP BY по имени пользователя.
Заворачиваем оконную функцию в подзапрос, чтобы отобрать пользователей с максимальным рангом.
*/
SELECT 
    username
FROM 
    (SELECT concat(trim(FROM uf.user_firstname), ' ', trim(FROM uf.user_lastname)) AS username, -- Очищаем от  возможных пробелов и объединяем имя и фамилию через пробел
    dense_rank () OVER (ORDER BY count(DISTINCT video_id) DESC) AS dr -- Вычисляем ранг пользователя без пропусков по кол-ву видео 
    FROM user_flags AS uf
    JOIN flag_review AS fr ON uf.flag_id = fr.flag_id
    WHERE lower(fr.reviewed_outcome) = 'approved' -- проиводим к нижнему регистру на случай разнобоя в данных по регистру
    GROUP BY username) AS user_rank
WHERE dr = 1;
```
 **Output**
|username|dr|
|:---|---:|
|Mark May|1|
|Richard Hasson|1|

```sql
/*
В нашей оконной функции рамка окна состоит из одной строки в виде агрегата количества уникальных роликов count(DISTINCT video_id).
Если организовать разделы с помощью PARTITION BY username, то в каждом разделе (для каждого польтзователя) ранг будет равен '1'.
Чтобы получить ранжированный список, расширяем раздел окна до всей таблицы, опуская в оконной функции PARTITION BY.
Для наглядности выведем результаты только оконной функции без джойнов и фильтров.
Этот запрос сначала формирует виртуальную таблицу из пользователей и количества их роликов.
Затем по ней в виде одного раздела вычисляется функция рамки окна dense_rank.

В этих результатах больше всего роликов у Марка, но в финальном запросе после наложения фильтртов по условиям задачи
появляется еще и Ричард, т.е. в датасете присутствует 2 человека с одинаковым максимальным количеством одобренных роликов.
*/
SELECT 
user_firstname,
count(video_id),
dense_rank () OVER (
	ORDER BY count(video_id) DESC
	)
FROM user_flags
GROUP BY user_firstname;

user_firstname|count |dense_rank|
--------------+------+----------+
Mark          |     6|         1|
Richard       |     5|         2|
Pauline       |     3|         3|
Gina          |     3|         3|
              |     3|         3|
Daniel        |     2|         4|
Greg          |     1|         5|
Evelyn        |     1|         5|
William       |     1|         5|
Loretta       |     1|         5|
Helen         |     1|         5|
Courtney      |     1|         5|
Mary          |     1|         5|
```
</details>
