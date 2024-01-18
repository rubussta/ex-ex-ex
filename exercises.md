## stratascratch.com (PostgreSQL)
<details>
<summary>Упражнение "Share of Active Users"</summary>
ID 2005<br>
<p>Output share of US users that are active. Active users are the ones with an "open" status in the table.</p>

Table: fb_active_users<br>
user_id: int<br>
name: varchar<br>
status: varchar<br>
country: varchar

 **Solution**
```sql
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
<summary>Упражнение "New Products"</summary>
ID 10318<br>
<p>You are given a table of product launches by company by year. 
Write a query to count the net difference between the number of products companies launched in 2020 with the number of products companies launched in the previous year. 
Output the name of the companies and a net difference of net products released for 2020 compared to the previous year.</p>

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
<summary>Упражнение "Premium Acounts"</summary>
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
-- Чтобы получить для решени ядве колонки, таблица объединяется сама с собой.
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
