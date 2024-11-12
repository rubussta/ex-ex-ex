## stratascratch.com (PostgreSQL)
<details>
<summary>Упражнение "Доля активных пользователей": приведение типов с использование условного или агрегатного выражения <br>#round() #count() #CASE_WHEN #FILTER</br></summary>
	
ID 2005  
	
"Share of Active Users"
Output share of US users that are active. Active users are the ones with an "open" status in the table.  

"Доля активных пользователей"
Выведите долю активных пользователей в США. Активные пользователи - это те, у кого в таблице статус "open".  

Table: fb_active_users  
user_id: int   
name: varchar  
status: varchar  
country: varchar  

 **Solution**

Чтобы формула "заработала" нужно поривести одно из значений к numeric.  
Вариант 1.
Использование условного выражения

```sql
SELECT
round(count(CASE WHEN status = 'open' THEN 1 ELSE NULL END) / count(user_id)::numeric, 1)  AS active_users_share
FROM fb_active_users;
```

Вариант 2.  
Использование агрегатного выражения

```sql
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
<summary>Упражнение "Новые продукты": сравнение периодов, выделенных из одной колонки таблицы с помощью CTE или подзапросов <br>#WITH_AS #JOIN</br></summary>

ID 10318  
	
"New Products"  
You are given a table of product launches by company by year. Write a query to count the net difference between the number of products companies launched in 2020 with the number of products companies launched in the previous year. Output the name of the companies and a net difference of net products released for 2020 compared to the previous year.

"Новые продукты"  
Вам предоставляется таблица запусков продуктов компаниями по годам. Напишите запрос, чтобы подсчитать чистую разницу между количеством продуктов, запущенных компаниями в 2020 году, и количеством продуктов, запущенных компаниями в предыдущем году. Выведите названия компаний и разницу в количестве продуктов, выпущенных за 2020 год, по сравнению с предыдущим годом.

Table: car_launches  

year: int  
company_name: varchar  
product_name: varchar  

 **Solution**

Вариант 1  

```sql
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
Вариант 2  

```sql
SELECT prod_2020.company_name, count(DISTINCT prod_2020.product_name) - count(DISTINCT prod_2019.product_name) AS net_products
    FROM 
       (SELECT  company_name, product_name, year
        FROM car_launches
        WHERE year = 2020) AS prod_2020
    JOIN 
       (SELECT  company_name, product_name, year
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
|Honda|-3|
|Jeep|1|
|Toyota|-1|

</details>
<details>
<summary>Упражнение "Премиальные аккаунты": добавление к таблице колонок на основе данных этой же таблицы <br>#LEFT_JOIN #count()</br></summary>  
	
ID 2097   
	
"Premium Acounts"  
You are given a dataset that provides the number of active users per day per premium account. 
A premium account will have an entry for every day that it’s premium. However, a premium account may be temporarily discounted and considered not paid, this is indicated by a value of 0 in the final_price column for a certain day. Find out how many premium accounts that are paid on any given day are still premium and paid 7 days later.  

Output the date, the number of premium and paid accounts on that day, and the number of how many of these accounts are still premium and paid 7 days later. 
Since you are only given data for a 14 days period, only include the first 7 available dates in your output.

Table: premium_accounts_by_day

account_id: varchar  
entry_date: datetime  
users_visited_7d: int  
final_price: int  
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
 
|entry_date|premium_paid_accounts|premium_paid_accounts_after_7d|
|---|---:|---:|
|2022-02-07|2|2|
|2022-02-08|3|2|
|2022-02-09|3|2|
|2022-02-10|4|3|
|2022-02-11|4|1|
|2022-02-12|3|2|
|2022-02-13|3|1|

</details>
<details>
<summary>Упражнение "Flags per Video": преобразование строковых данных с условным выражением и очисткой от возможных пробелов<br>#count() #DISTINCT #concat() #trim()</br></summary>  
	
ID 2102  
	
For each video, find how many unique users flagged it. A unique user can be identified using the combination of their first name and last name. Do not consider rows in which there is no flag ID.</p>
<p>Для каждого видео найдите, сколько уникальных пользователей отметили его. Уникального пользователя можно идентифицировать, используя комбинацию его имени и фамилии. Не рассматривайте строки, в которых нет идентификатора флага.

Table: user_flags<br>
user_firstname: varchar<br>
user_lastname: varchar<br>
video_id: varchar<br>
flag_id: varchar

 **Solution**

Функция concat объединяет строки для формирования id. Условное выцражение COALESCE возвращает пустую строку, если нет имени или фамилии.  Функция trim удаляет пробелы слева и справа.

```sql
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
<summary>Упражнение "User with Most Approved Flags": оконная функция без PARTITION BY<br>#dense_rank() #OVER #concat() #trim()</br></summary>
	
ID 2104  
	
Which user flagged the most distinct videos that ended up approved by YouTube? Output, in one column, their full name or names in case of a tie. In the user's full name, include a space between the first and the last name.  
Какой пользователь отметил наиболее отличающиеся друг от друга видеоролики, которые в итоге были одобрены YouTube? Выведите в одном столбце их полное имя или фамилии в случае совпадения. В полном имени пользователя укажите пробел между именем и фамилией.  
	
Table: user_flags  

user_firstname: varchar  
user_lastname: varchar  
video_id: varchar  
flag_id: varchar  

Table: flag_review  

flag_id: varchar  
reviewed_by_yt: bool  
reviewed_date: datetime  
reviewed_outcome: varchar  

 **Solution**

По условию задания фактически нужно получить пользователя (или пользователей), у которого было больше всех уникальных видео, которые были одобрены к публикации. Таблица flag_review отражает результат одоброения.
Очевидно, что будет неизвестное кол-во пользователей с одинаковым числом роликов и нам неизвестен аргумент для LIMIT в связке с ORDER BY. Поэтому, пользователей нужно проранжировать и получить список тех, у кого наивысший ранг. Также вызывает вопрос идентификация пользователей по имени и фамилии по условию задачи. Но ID пользователя нет и используем, что есть. 

Объединяем таблицы и отфильтровыввем строки с пользователями, у которых видео одобрены. Для ранжирования пользоватей используем оконную функцию dense_rank без PARTITION BY. Без PARTITION BY строки раздела состоят из всех строк таблицы, которые группируются с помощью GROUP BY по имени пользователя. Заворачиваем оконную функцию в подзапрос, чтобы отобрать пользователей с максимальным рангом.

```sql
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

В нашей оконной функции рамка окна состоит из одной строки в виде агрегата количества уникальных роликов count(DISTINCT video_id).
Если организовать разделы с помощью PARTITION BY username, то в каждом разделе (для каждого польтзователя) ранг будет равен '1'.
Чтобы получить ранжированный список, расширяем раздел окна до всей таблицы, опуская в оконной функции PARTITION BY.
Для наглядности выведем результаты только оконной функции без джойнов и фильтров.
Этот запрос сначала формирует виртуальную таблицу из пользователей и количества их роликов.
Затем по ней в виде одного раздела вычисляется функция рамки окна dense_rank.

В этих результатах больше всего роликов у Марка, но в финальном запросе после наложения фильтртов по условиям задачи
появляется еще и Ричард, т.е. в датасете присутствует 2 человека с одинаковым максимальным количеством одобренных роликов.  

```sql
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
<details>
<summary>Упражнение "Найдите студентов с медианным баллом": использование агрегатного выражения с медианой в WHERE<br>#percentile_disc()</br></summary>

ID 9610  

"Find students with a median writing score"  
Output ids of students with a median score from the writing SAT.  
Выведите идентификаторы учащихся со средним баллом по результатам письменного экзамена SAT.  

Table: sat_scores  

(school varchar),  
(teacher varchar),  
(student_id float),  
(sat_writing float),    
(sat_verbal float),  
(sat_math float),  
(hrs_studied float),  
(id int),  
(average_sat float)

**Solution**

Для вычисления медианы в PostgreSQL используются сортирующие агрегатные функции 
percentile_disc или percentile_cont, возвращающие дискретный или непрерывный процентиль.

```sql
SELECT student_id, sat_writing
FROM sat_scores
WHERE sat_writing = (SELECT percentile_disc(0.5) WITHIN GROUP (ORDER BY sat_writing)
    FROM sat_scores);

```
Почему агрегатная функция выведена в подзапрос?
Потому что в условии фильтрации она становится агрегатным выражением и согласно документации:
"Агрегатное выражение может фигурировать только в списке результатов или в предложении HAVING команды SELECT. 
Во всех остальных предложениях, например WHERE, они запрещены, так как эти предложения логически вычисляются до того, 
как формируются результаты агрегатных функций."

 **Output**
 
|student_id|sat_writing|
|---:|---:|
|100|512|
|109|512|
|113|512|
</details>
<details>
<summary>Упражнение "Классификация бизнесов": поиск слова в тексте по шаблону<br>#CASE #POSIX #@@</br></summary>  

ID 9726  

"Classify Business Type"  
Classify each business as either a restaurant, cafe, school, or other.  
- A restaurant should have the word 'restaurant' in the business name.  
- A cafe should have either 'cafe', 'café', or 'coffee' in the business name.  
- A school should have the word 'school' in the business name.  
- All other businesses should be classified as 'other'.
Output the business name and their classification.

Table: sf_restaurant_health_violations  

(business_id int),  
(business_name varchar),  
(business_address varchar),  
(business_city varchar),  
(business_state varchar),  
(business_postal_code float),  
(business_latitude float),  
(business_longitude float),  
(business_location varchar),  
(business_phone_number float),  
(inspection_id varchar),  
(inspection_date datetime),  
(inspection_score float),  
(inspection_type varchar),  
(violation_id varchar),  
(violation_description varchar),  
(risk_category varchar)  

 **Solution**
 
По условию задачи имеем строку из нескольких слов, в которой нам нужно найти определенное слово в его конкретной форме: "...should have the word 'restaurant'..."  и результатом этого поиска должно быть логическое значение, которое потом бужети участвовать в условном операторе CASE для формирования новой классификации строк.  

Особенность заключается в том, что в задании явно указано, что нужно найте не строку, а слово, которое может быть в начеле строки, в середине либо в конце, т.е нужно составить такой шаблон, который будет учитывать расположение пробелов.  

Начнем с варианта поиска в строке по шаблону. Шаблон будем составлять из регулярных выражений POSIX со спецсимволами и спецкодами регистра символов и начала/конца слова. Для краткости возьмем из задания только слово 'cafe' и смоделируем варианты его нахождения в строке.  
 
```sql
SELECT 
'1. Allstars Cafe Inc' ~* '(\mcafe\M)',
'2. Cafe Online' ~* '(\mcafe\M)',
'3. Luna Cafe' ~* '(\mcafe\M)',
'4. Cafemania Shope' ~* '(\mcafe\M)',
'5. InCafe' ~* '(\mcafe\M)';

?column?|?column?|?column?|?column?|?column?|
--------+--------+--------+--------+--------+
true    |true    |true    |false   |false   |
```

Шаблон можно подкорректировать с учетом формы слов. Этого нет в задании, но в датасете есть такие наименования организаций, для которых это имеет смысл.  

```sql
SELECT 
'6. Gateway High/Kip Schools' ~* '\mschool.*',
'7. Restaurante Montecristo' ~* '\mrestaurant.*';

?column?|?column?|
--------+--------+
true    |true    |

```

**1 Вариант решения**

```sql
SELECT 
    business_name,
    CASE WHEN business_name ~* '\mrestaurant.*' THEN 'restaurant'
        WHEN business_name ~* '(\mcafe\M|\mcafé\M|\mcoffee\M)' THEN 'cafe'
        WHEN business_name ~* '\mschool.*' THEN 'school'
        ELSE 'other'
        END AS business_type
FROM sf_restaurant_health_violations
LIMIT 5;

```

**Output 1**

|business_name|business_type|
|---|---|
|John Chin Elementary School|school|
|Sutter Pub and Restaurant|restaurant|
|SRI THAI CUISINE|other|
|Washington Bakery & Restaurant|restaurant|
|Brothers Restaurant|restaurant|

Подсчитаем количество категорий в датасете, которые мы сформировали с помощью условного оператора.

```sql
WITH bt AS (
    SELECT 
    business_name,
    CASE WHEN business_name ~* '\mrestaurant.*' THEN 'restaurant'
        WHEN business_name ~* '(\mcafe\M|\mcafé\M|\mcoffee\M)' THEN 'cafe'
        WHEN business_name ~* '\mschool.*' THEN 'school'
        ELSE 'other'
        END AS business_type
    FROM sf_restaurant_health_violations) 
SELECT  business_type, count(business_type)
FROM bt
GROUP BY business_type;

```

|business_type|count|
|:---|---:|
|school|7|
|restaurant|38|
|cafe|50|
|other|202|

**2 Вариант решения**   

Применим функции полнотекстового поиска. Слова в тексте будут нормализованы разбором на фрагменты и лексемы. Это позволит в нашем случае сопоставить 'schools' и 'school'. Укажем конфигурацию 'english' текстового поиска явно в функции. Для итальянских названий рестаранов в to_tsquery применим префикс :*

```sql
SELECT DISTINCT business_name, 
    CASE 
        WHEN to_tsvector('english', business_name) @@ to_tsquery('english', 'school') THEN 'school' 
        WHEN to_tsvector('english',  business_name) @@ to_tsquery('english', 'restaurant:*') THEN 'restaurant' 
        WHEN to_tsvector('english',  business_name) @@ to_tsquery('english', 'cafe | café | coffee') THEN 'cafe' 
        ELSE 'other' 
        END AS business_type 
FROM sf_restaurant_health_violations
LIMIT 5;

```

 **Output 2**

|business_name|business_type|
|---|---|
|ABSINTHE PASTRY|other|
|Akira Japanese Restaurant|restaurant|
|AK SUBS|other|
|A La Turca|other|
|Allstars Cafe Inc|cafe|

Посмотрим на результат классификации:
```sql
WITH bt AS (
    SELECT  business_name,
    CASE 
        WHEN to_tsvector('english', business_name) @@ to_tsquery('english', 'school') THEN 'school' 
        WHEN to_tsvector('english',  business_name) @@ to_tsquery('english', 'restaurant:*') THEN 'restaurant' 
        WHEN to_tsvector('english',  business_name) @@ to_tsquery('english', 'cafe | café | coffee') THEN 'cafe' 
        ELSE 'other' 
        END AS business_type 
    FROM sf_restaurant_health_violations) 
SELECT  business_type, count(business_type)
FROM bt
GROUP BY business_type;

```

|business_type|count|
|:---|---:|
|school|7|
|restaurant|38|
|cafe|50|
|other|202|  

**3 Вариант решения**   

Сравнение табличных строк и массивов.

```sql
SELECT 
     business_name,
    CASE WHEN business_name ~* ANY (ARRAY['\mrestaurant.*']) THEN 'restaurant'
        WHEN business_name ~* ANY (ARRAY['\mcafe\M', '\mcafé\M', '\mcoffee\M']) THEN 'cafe'
        WHEN business_name ~* ANY (ARRAY['\mschool.*']) THEN 'school'
        ELSE 'other'
        END AS business_type
FROM sf_restaurant_health_violations;

```

Output аналогичен предыдущим.
</details>
<details>
<summary>Упражнение "Самая высокая зарплата по департаменту": оконная функция с фильтрацией по ее результатам<br>#rank() #OVER</br></summary>  

ID 9897  

"Highest Salary In Department"  
Find the employee with the highest salary per department.  
Output the department name, employee's first name along with the corresponding salary.  

Table:  employee  
(id int),  
(first_name varchar),  
(last_name varchar),  
(age int),  
(sex varchar),  
(employee_title varchar),  
(department varchar),  
(salary int)  

**Solution**
 
Для определения максимальной зарплаты используем оконную функцию rank() и заворачиваем ее в подзапрос, чтобы использовать для фильтрации в основном запросе. В данном случае агрегатные функции min()/max() не подходят, поскольку они возвращают единственное значение, но у нескольких человек могут быть одинаковые зарплаты.
 
```sql
SELECT department, first_name, salary
FROM -- Подзапрос для фильтрации по значению rank 
    (SELECT department, first_name, salary, 
    rank() OVER (PARTITION BY department ORDER BY salary DESC) AS r
    FROM employee) AS rr
WHERE r = 1
ORDER BY salary DESC;

```

 **Output**
 
|department|first_name|salary|
|---|---|---:|
|Management|Richerd|250000|
|Sales|Mick|2200|
|Audit|Shandler|1100|
</details>
<details>
<summary>Упражнение "Наивысшие показатели по менеджерам": оконная функция без PARTITION BY с форматированием строк<br>#rank() #OVER #concat() #initcap() #trim()</br></summary>

ID 9905   

"Highest Target Under Manager"  
Find the highest target achieved by the employee or employees who works under the manager id 13. Output the first name of the employee and target achieved. The solution should show the highest target achieved under manager_id=13 and which employee(s) achieved it.  
	
Table: salesforce_employees  

(id int),  
(first_name varchar),  
(last_name varchar),  
(age int),  
(sex varchar),  
(salary int),  
(target int),  
(manager_id int)  

**Solution**
 
Чтобы найти максимальное целевое значение используем оконную функцию rank() в подзапросе. Поскольку нужны подчиненные только одного менеджера с ID=13, то здесь же оставляем только его. В этом случае нет необходимости выделять рамки окна среди менеджеров, он все равно один, и мы опускаем в синтаксисе оконной функции PARTITION BY, расширяя рамку окна до всего раздела, который состояит у нас из работников одного менеджера.  

Результат возвращаем с форматированием строк. В исходных строках функцией trim() образаем пробелы, если они там были. Имя и фамилию объединяем через пробел с помощью строковой функции concat(), предварительно приведя их к нижнему регистру с заглавной буквы в начале слова с помощью initcap().
 
```sql
SELECT 
concat(initcap (trim(first_name)), ' ', initcap (trim(last_name))) AS employee,
target
FROM
    (SELECT first_name, last_name, target, manager_id,
    rank() OVER (ORDER BY target DESC) AS r
    FROM salesforce_employees
    WHERE manager_id = 13
    ) AS ranked_target
WHERE r = 1;

```

 **Output**
 
|employee|target|
|---|---:|
|Nicky Bat|400|
|Steve Smith|400|
|David Warner|400|
</details>
<details>
<summary>Упражнение "Поиск TOP10 песен в 2010 году": исключаем повторы с помощью группировки<br>#GROUP BY</br></summary>  
	
ID 9650  

"Find the top 10 ranked songs in 2010"  
What were the top 10 ranked songs in 2010? Output the rank, group name, and song name but do not show the same song twice. Sort the result based on the year_rank in ascending order.   

Table: llboard_top_100_year_end    

(year int),  
(year_rank int),  
(group_name varchar),   
(artist varchar),  
(song_namey varchar),  
(id int)  

**Solution**
 
```sql
SELECT year_rank, group_name, song_name
FROM billboard_top_100_year_end
WHERE year = 2010 AND (year_rank >0 AND year_rank <11)
GROUP BY  1, 2, 3
ORDER BY year_rank ASC;

```

 **Output**
 
|year_rank|group_name|song_name|
|---:|---|---|
|1|Ke$ha|TiK ToK|
|2|Lady Antebellum|Need You Now|
|3|Train|Hey, Soul Sister|
|4|Katy Perry feat. Snoop Dogg|California Gurls|
|5|Usher feat. will.i.am|OMG|
|6|B.o.B feat. Hayley Williams|Airplanes|
|7|Eminem feat. Rihanna|Love The Way You Lie|
|8|Lady Gaga|Bad Romance|
|9|Taio Cruz|Dynamite|
|10|Taio Cruz feat. Ludacris|Break Your Heart|
</details>
<details>
<summary>Упражнение "Количество нарушений": группировка по году в дате и расчет агрегата<br>#date_part() #count()</br></summary>  
	
ID 9728  

"Number of violations"
You're given a dataset of health inspections. Count the number of violation in an inspection in 'Roxanne Cafe' for each year. If an inspection resulted in a violation, there will be a value in the 'violation_id' column. Output the number of violations by year in ascending order.   

Table: sf_restaurant_health_violations    

(business_id int),  
(business_name varchar),   
(violation_id varchar),  
(inspection_date datetime)  

**Solution**
 
```sql
SELECT 
date_part ( 'year', inspection_date ) AS year,
count(violation_id) AS n_inspections
FROM sf_restaurant_health_violations
WHERE business_name = 'Roxanne Cafe'
GROUP BY year
ORDER BY year ASC;

```

 **Output**
 
|year|n_inspections|
|---|---:|
|2015|5|
|2016|2|
|2018|3|
</details>
<details>
<summary>Упражнение "Find the rate of processed tickets for each type": подсчет доли по колонке с условным оператором<br>#CASE #sum() #count()</br></summary>  
	
ID 9781   

"Find the rate of processed tickets for each type"   

Table: facebook_complaints    

(complaint_id int),  
(type int),   
(processed bool)  

**Solution**

Преобразуем булевы значения с помощью условного выражения CASE в упроженной форме синтаксиса и применяем к ним агроегатную функцию. В конструкции CASE опущен ELSE. В нашем случае все, кроме TRUE принимает значание NULL. В знаменателе формулы подсчитываем количество непустых значений в processed. Но если нам нужна доля значений с учетом NULL, то можно поделить все на  count(*), т.е. на количество строк в таблице.  

```sql
SELECT type, 
sum(CASE processed WHEN TRUE THEN 1 END)::numeric / count(processed) AS processed_rate
FROM facebook_complaints
GROUP BY type
ORDER BY type;

```

 **Output**
 
|type|processed_rate|
|---|---:|
|0|0.667|
|1|0.667|
</details>
<details>
<summary>Упражнение "Доход от клиентов в марте": подсчет суммы с группировкой и фильтрацией по дате<br>#EXTRACT(field FROM source)</br></summary>

ID 9782    

"Customer Revenue In March"  
Calculate the total revenue from each customer in March 2019. Include only customers who were active in March 2019.Output the revenue along with the customer id and sort the results based on the revenue in descending order.   

Table: orders   

(id int),  
(cust_id int),  
(order_date datetime),  
(order_details varchar),  
(total_order_cost int) 

**Solution**

С помощью функции даты EXTRACT(field FROM source) отфильтровываем клиентов с заказами в марте 2019 года, группируем эти заказы по id клиентов и вычисляем агрегатную функциюю суммирования выручки по каждому id. Полученные суммы упорядочиваем по убыванию. 

```sql
SELECT cust_id, sum(total_order_cost) AS revenue
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2019 AND EXTRACT(MONTH FROM order_date) = 3
GROUP BY cust_id
ORDER BY revenue DESC;

```

 **Output**
 
|cust_id|revenue|
|---|---:|
|3|210|
|15|150|
|7|80|
|12|20|
</details>
<details>
<summary>Упражнение "Подсчет количества упоминаний слов в документах": cбор статистики по документу <br>#ts_stat #to_tsvector() #ILIKE</summary>

ID 9817    
	
"Find the number of times each word appears in drafts".  
Output the word along with the corresponding number of occurrences.  

Table: google_file_store   

(filename varchar),  
(contents varchar) 

**Solution**

Для подсчета количестива слов в документах, которые представлены в данном случае в виде двух строк в таблице - названия файла и собственно текста документа, используем функцию сбора статистики по документам ts_stat. Функция принимает текст в формате tsvector в виде одной строки. Исходный текстовой формат преобразуем в tsvector с помощью функции to_tsvector. Возвращаемая статистика будет разной в зависимости от того, как мы определили формат tsvector и от конфигурации текстового поиска сервера. Поскольку нам нужны не все документы, то выбираем по шаблону ILIKE нужные.

**1 Вариант решения**

Преобразуем исходный текст документов в формат tsvector функцией to_tsvector с настройками текстового поиска на сервере по-умолчанию.

```sql
SELECT word, nentry
FROM ts_stat('SELECT to_tsvector(contents) FROM google_file_store WHERE filename ILIKE ''draft%''')
ORDER BY nentry DESC;

```

 **Output 1**
 По-умолчанию на данном сервере нормализация текста не предполагает удаление стоп-слов с виде артиклей. Они попадают в статистику как отделоьные лексемы.
 
|word|nentry|
|---|---:|
|a|3|
|market|3|
|many|2|
|which|2|
|the|2|
|stock|2|
|predicts|2|
|of|2|
|would|2|
|make|2|
|investors|2|
|happy|2|
|exchange|2|
|bull|2|
|bear|1|
|awaiting|1|
|are|1|
|in|1|
|and|1|
|too|1|
|fact|1|
|that|1|
|analysts|1|
|but|1|
|possibility|1|
|optimism|1|
|we|1|
|much|1|
|warn|1|

**2 Вариант решения**

В функции to_tsvector явно указываем словарь english.

```sql
SELECT word, nentry
FROM ts_stat('SELECT to_tsvector(''english'', contents) FROM google_file_store WHERE filename ILIKE ''draft%''')
ORDER BY nentry DESC;

```

 **Output 1**
 Результат разбора текста другой. Выделены лексемы без окончпний, стоп-слова отброшены, статистика отличная от конфигурации по-умолчанию.
 
|word|nentry|
|---|---:|
|market|3|
|stock|2|
|predict|2|
|would|2|
|mani|2|
|make|2|
|investor|2|
|happi|2|
|exchang|2|
|bull|2|
|analyst|1|
|bear|1|
|possibl|1|
|optim|1|
|much|1|
|warn|1|
|fact|1|
|await|1|
</details>
<details>
<summary>Упражнение "Количество выживших и не выживших на Титанике в разрезе класса билетов": разбиение одной колонки на несколько по условному выражению <br>#CASE</br></summary>  
	
ID 9881  

"Make a report showing the number of survivors and non-survivors by passenger class".  	
Make a report showing the number of survivors and non-survivors by passenger class. Classes are categorized based on the pclass value as:
pclass = 1: first_class  
pclass = 2: second_classs  
pclass = 3: third_class  
Output the number of survivors and non-survivors by each class.

Table:  titanic   

(passengerid int),  
(survived int),   
(pclass int)  

**Solution**

С помощью условного выражения CASE из одной и той же строки выделяем подвыборки и расчитывам в них агрегат count по каждой группе survived.  

```sql
SELECT
survived, 
count(CASE pclass WHEN  1 THEN 1 ELSE 0 END) AS first_class,
count(CASE pclass WHEN  2 THEN 2 ELSE 0 END) AS second_classss,
count(CASE pclass WHEN  3 THEN 3 ELSE 0 END) AS third_class
FROM titanic
GROUP BY survived;

```

 **Output**

|survived|first_class|second_classss|third_class|
|---|---:|---:|---:|
|0|11|6|42|
|1|10|12|19|
</details>
<details>
<summary>Упражнение "Вторая зарплата после самой высокой": ранжирование с оконной функцией <br>#dense_rank() #OVER</br></summary>  
	
ID 9892    

"Second Highest Salary"
Find the second highest salary of employees.

Table:  employee   

(id int),  
(name varchar),   
(salary int)  

**Solution**

Вызываем оконную функция плотного ранга dense_rank () по всему разделу без указания рамки окна, чтобы получить второй по убывания ранг дохода работников. Поскольку у нескольких работников может быть ожинаковый доход, а нам нужно одно значение, то применяем оператор DISTINCT.  
Также по причине того, что может быть несколько одинаковых значений доходов, запрос получается несколько сложнее, чем просто сортировка ORDER BY salary desc LIMIT 1 OFFSET 1.

```sql
SELECT DISTINCT salary
FROM (SELECT salary, 
        dense_rank() OVER (ORDER BY salary DESC) AS rnk 
        FROM employee) AS empl_rank
WHERE rnk = 2;

```

 **Output**

|salary|
|---|
|200000|
</details>
<details>
<summary>Упражнение "Зарплата работников и их менеджеров": объединение таблицы самой с собой<br>#JOIM</br></summary>

ID 9894  

"Employee and Manager Salaries"  
Find employees who are earning more than their managers. Output the employee's first name along with the corresponding salary.

Table:  employee   

(id int),  
(first_name varchar),  
(last_name varchar),  
(salary int),   
(manager_id int)  

**Solution 1**

Чтобы сопоставить зарплаты работников и менеджеров, объединяем таблицу саму с собой, используя алиасы. В итоговой таблице каждому сотруднику сопоставляется его менеджер и появляется возможность сравнить их зарплаты.

```sql
SELECT
    emp.first_name, 
    emp.salary AS employee_salary,
    mgr.salary AS manager_salary
FROM employee AS emp 
JOIN employee AS mgr ON  emp.manager_id = mgr.id 
WHERE emp.salary > mgr.salary;
```
**Solution 2**

Другой более объемный по коду способ - с помощью CTE сгенерировать отдельную таблицу менеджеров и уже ее сравнивать с таблицей работников 

```sql
WITH manager AS ( --Таблица зарплат менеджеров
    SELECT manager_id, salary
    FROM employee
    WHERE id = manager_id
    )
SELECT first_name, emp.salary as employee_salary, mgr.salary AS manager_salary
FROM employee AS emp --Таблица зарплат работников
JOIN manager AS mgr ON emp.manager_id = mgr.manager_id
WHERE emp.salary > mgr.salary
ORDER BY emp.salary DESC;
```

 **Output**

|first_name|employee_salary|manager_salary|
|---|--:|--:|
|Richerd|250000|200000|
</details>
<details>
<summary>Упражнение "Самы большие заказы": JOIN таблиц в CTE и вложенный SELECT для сложной фильтрации<br>#LEFT JOIN #WITH</br></summary>  
	
ID 9915  
	
"Highest Cost Orders"	
Find the customer with the highest daily total order cost between 2019-02-01 to 2019-05-01. If customer had more than one order on a certain day, sum the order costs on daily basis. Output customer's first name, total cost of their items, and the date. For simplicity, you can assume that every first name in the dataset is unique.

Table:  customers

(id int),  
(first_name varchar),  
(last_name varchar),  
(city varchar),  
(address varchar),  
(phone_number varchar) 

Table: orders  

(id int),  
(cust_id int),  
(order_date datetime),  
(order_details varchar),  
(total_order_cost int)  

**Solution**

Сложное условие фильтрации разбиваем на две задачи. Сначала в CTE отбыраем клиентов с максимальной суммой заказов в указанном интервале времени. Заказы одних и тех же клиентов с одинаковой суммой суммируются. Здесь же с помощью JOIN присоединяем к заказам имена клиентов. Затем из построенной в CTE "виртуальной" таблицы отбираем заказ с максимальной суммой.

```sql
WITH cast_orders AS (
    SELECT cust_id,
        first_name, 
        sum(o.total_order_cost) AS total_order_cost, 
        order_date
    FROM orders AS o
    LEFT JOIN customers AS c ON o.cust_id = c.id
    WHERE o.order_date >= '2019-02-01' AND o.order_date <= '2019-05-01'
    GROUP BY first_name, cust_id, order_date
    )
SELECT first_name, total_order_cost, order_date
FROM cast_orders
WHERE total_order_cost = (SELECT max(total_order_cost) FROM cast_orders);
```

 **Output**

|first_name|total_order_cost|order_date|
|---|--:|---|
|Jill|275|2019-04-19|
</details>
<details>
<summary>Упражнение "Самые большие Олимпийские игры": поиск максимума с ранжированием уникальных строк<br>#GROUP BY #ORDER BY #HAVING</br></summary>  
	
ID 9942  
	
"Largest Olympics"  	
Find the Olympics with the highest number of athletes. The Olympics game is a combination of the year and the season, and is found in the 'games' column. Output the Olympics along with the corresponding number of athletes.

Table:  olympics_athletes_events  

id: int  
name: varchar  
sex: varchar  
age: float  
height: float  
weight: datetime  
team: varchar  
noc: varchar  
games: varchar  
year: int  
season: varchar  
city: varchar  
sport: varchar  
event: varchar  
medal: varchar  

**Solution**

В задании не оговорено, что нужно объединять зимние и летние олимпийские игры, поэтому будеи считать, что в колонке games указаны те самые игры, участников которых нужно подсчитать и найти максимум. Для этого можно сделать группировку по games и подсчитываем с помощью агрегата count() количество спортсменов по их ID. Однако, здесь со стороны датасета кроется сюрприх. ID оказываются не уникальными в рамках одних игр.

Смотрим уникальность ID и видим, что есть дублирования.

```sql
SELECT games, name, count(id)
FROM olympics_athletes_events
GROUP BY games, name
HAVING count(id) > 1
ORDER BY count(id) DESC
LIMIT 5;
```
 
|games|name|count|
|---|---|--:|
|1908 Summer|Samuel Sam Blatherwick|3|
|1904 Summer|Albertson Van Zo Post|2|
|1908 Summer|Leslie George Rich|2|
|1924 Summer|Pierre Tolar|2|
|1992 Winter|Ole Kristian Furuseth|2
|1994 Winter|Christine Jacoba Aaftink|2|

Используем DISTINCT для отбора уникальных строк таблицы. Для определения максимального количества спортсменов среди прошедших игр можно поступить ннесколькими способами. Поскольку в PostgreSQL отсутствует функция TOP, то мы сортируем строки по убыванию и оставляем самую верхнюю. Также можно сформировать таблицу в CTE и затем уже в ней искать максимум с помощью функции max(). 

В данном датасете арят ли может быть ситуация, что вх играх в разные годы принимало участие одинаковое кол-во участников. Но если предусмотреть и такую возмозможность, то в качестве финальной агрегатной функции можно использовать плотный ранг dense_rank() в оконной функции и тогда, если было несколько одинаковых максимальных чисел, то у них у всех будет ранг = 1. 

```sql
SELECT games, count(DISTINCT id) AS athletes_count
FROM olympics_athletes_events
GROUP BY games
ORDER BY count(id) DESC
LIMIT 1;
```

 **Output**

|games|athletes_count|
|---|--:|
|1924 Summer|118|
</details>
<details>
<summary>Упражнение "Top Ranked Songs": группировка с агрегатом и ранжированием (#GROUP BY#count#ORDER BY)</summary>
<br><p>ID 9991</p>  
	
Find songs that have ranked in the top position. Output the track name and the number of times it ranked at the top. Sort your records by the number of times the song was in the top position in descending order.

Table: spotify_worldwide_daily_song_ranking

id: int  
position: int  
trackname: varchar  
artist: varchar 
streams: int 
url: varchar  
date: datetime  
region: varchar   

**Solution**

На первом этапе для ускорения запроса фильтруем терки с рейтингом = 1 и затем делаем группировку по треку и подсчитываем количество попаданий в топы, и сортируем это количество по убыванию.

```sql
SELECT trackname, count(position) AS times_top1
FROM spotify_worldwide_daily_song_ranking
WHERE position = 1
GROUP BY trackname
ORDER BY times_top1 DESC;
```

 **Output**

|trackname|times_top1|
|---|--:|
|HUMBLE.|7|
|Bad and Boujee (feat. Lil Uzi Vert)|1|
|Look What You Made Me Do|1|
</details>
<details>
<summary>Упражнение "Результаты выборов": вложкнные подзапросы с оконными функциями <br>#OVER #cdense_rank() #round()</br></summary>
<br><p>ID 2099</p>  
"Election Results"	
The election is conducted in a city and everyone can vote for one or more candidates, or choose not to vote at all. Each person has 1 vote so if they vote for multiple candidates, their vote gets equally split across these candidates. For example, if a person votes for 2 candidates, these candidates receive an equivalent of 0.5 vote each.
Find out who got the most votes and won the election. Output the name of the candidate or multiple names in case of a tie. To avoid issues with a floating-point error you can round the number of votes received by a candidate to 3 decimal places.

Table:  voting_results

voter: varchar  
candidate: varchar 

**Solution**

Сначала в окне с избирателями расчитываем долю кандидата в бюллетене. Оборачиваем это в подзапрос и по сгруппированным кандидатам расчитываем в оконной функции сумму долей кандидата и плотный ранг результ атов голосования. Чтобы по условию задачи возвратить победителя голосования, оборачиваем все это еще в один подзапрос и отфильтровываем строки с нужным рангом. Для наглядности берем три первых места.


```sql
SELECT  candidate, total_vote_score, place
    FROM
        (SELECT
           candidate,
           round(sum(vote_num), 3) AS total_vote_score, -- Итоговая сумма очков кандидата
           dense_rank() OVER (ORDER BY round(sum(vote_num), 3) DESC) AS place
        FROM
            (SELECT 
                voter,
                candidate,
                1.0 / count(*) OVER (PARTITION BY voter) AS vote_num -- Доля кандидата в бюллетене
            FROM voting_results
            WHERE candidate IS NOT NULL) AS candidate_score -- Исключаем неголосовавших
        GROUP BY candidate) AS vote_winer
WHERE place < 4; -- тройка победителей голосования

```

 **Output**

|candidate|total_vote_score|place|
|---|--:|--:|
|Christine|5.283|1|
|Ryan|5.15|2|
|Nicole|2.7|3|
</details>
<details>
<summary>Упражнение "Самые высокодоходные компании": вложкнные подзапросы с оконной функцией ранжирования #OVER #dense_rank() #GROUP_BY #ORDER_BY </summary>

ID 10354   
	
"Most Profitable Companies"  
Find the 3 most profitable companies in the entire world. Output the result along with the corresponding company name. Sort the result based on profits in descending order.

Table:  forbes_global_2010_2014s

company: varchar  
sector: varchar  
industry: varchar  
continent: varchar  
country: varchar  
marketvalue: float  
sales: float  
profits: float  
assets: float  
rank: int  
forbeswebpage: varchar  

**Solution**

Из имени таблицы датасета понятно, что жанные там содержаться за несколько лет и часть компаний должна дублироваться. Поэтому группируем компании по название и считаем сумму прибыли за все годы. Оборачивам эту таблицу в подзапрос и по ней считаем плотный ранг danse_rank() для построения рейтинга по суммарной прибыли. Эту конструкцию также оболрачиваем в подзапрос для отбора ТОП-3 компаний по рангу. danse_rank() можно просто заменить ранжированием с лимитом на 3 записи, поскольку наврят ли прибыль будет одинаковой у двух компаний, но поскольку в датасете низкая точность этого атрибута, то используем плотный ранг на случай, если в  ТОП-3 окажется 4 компании. 


```sql
SELECT company, profit
FROM (
    SELECT company, profit, dense_rank() OVER ( ORDER BY profit DESC) AS r
    FROM (
        SELECT  company, sum(profits) AS profit
        FROM forbes_global_2010_2014
        GROUP BY company
        ) AS company_list -- Список компаний с суммарной прибылью
    ) AS r_comp_list -- Список ранжированных компаний
WHERE r <= 3
ORDER BY profit DESC;

```

 **Output**

|company|profit|
|---|--:|
|ICBC|42.7|
|Gazprom|39|
|Apple|37|
</details>
<details>
<summary>Упражнение "Работники с самыми высокими зарплатами": оконная функция ранжирования либо условное выражение <br>#OVER #dense_rank() #CASE</br></summary>

ID 10353   

"Workers With The Highest Salaries"  
You have been asked to find the job titles of the highest-paid employees. Your output should include the highest-paid title or multiple titles with the same salary.

Table: worker, title  

worker_id: int  
first_name: varchar  
last_name: varchar  
salary: int  
joining_date: datetime  
department: varchar  

Table: title  

worker_ref_id: int  
worker_title: varchar  
affected_from: datetime  

**Solution 1**

Объединяем две таблицы в одну через внутреннее соединение JOIN. Полученную общую таблицу ранжируем по зарплатам в оконной функции плотного ранга  dense_rank() и затем из полученного результата отбираем работников с наивысшим  рангом по зарплатам. Поскольку функция ранжирования является плотной, то одинаковым зарплатам присваивается одинаковый ранг без пропуска очередности. 

```sql
SELECT
worker_title AS best_paid_title 
FROM
    (SELECT 
        worker_title, 
        salary,
        dense_rank() OVER (ORDER BY salary DESC) AS salary_rank -- Ранг работника по зарплате
    FROM worker AS w
    JOIN title AS t ON w.worker_id = t.worker_ref_id) AS ws -- Общий список работников с долджностями и зарплатами
WHERE salary_rank = 1;

```

**Solution 2**

После объединения таблиц используем условное выражение CASE и агрегатную функцию max() в подзапросе для переопределения типа отношения по нужному нам условию максимальных зарплат и затем возвращаем эти строки нужного нам типа, отсекая ненужные с типом NULL.

```sql
SELECT best_paid_title
FROM
    (SELECT 
        CASE WHEN salary = (SELECT max(salary) FROM worker) THEN worker_title 
        END AS best_paid_title
    FROM worker AS w
    JOIN title AS t ON w.worker_id = t.worker_ref_id) AS ws
WHERE best_paid_title IS NOT NULL;

```

 **Output**

|best_paid_title|
|---|
|Manager|
|Asst. Manager|
</details>
<details>
<summary>Упражнение "Средняя прододжительность сессии пользователя": CTE, оконная функция, объединение таблицы самой с собой <br>#WITH_AS #OVER #row_number() #JOIN</br></summary>

ID 10352  

"Users By Average Session Time"  
Calculate each user's average session time. A session is defined as the time difference between a page_load and page_exit. For simplicity, assume a user has only 1 session per day and if there are multiple of the same events on that day, consider only the latest page_load and earliest page_exit, with an obvious restriction that load time event should happen before exit time event . Output the user_id and their average session time.

Table:  facebook_web_log

user_id: int  
timestamp: datetime  
action: varchar  

**Solution**

Метрика поведения пользователя Average Session Time или Average Session Duration (ASD) - средняя продолжительность сессии. На первом этапе в общем табличном выражении формируем пронумерованный упорядоченный список действий пользователя, состоящий из открытия и закрытия сессий. Эту нумерацию будем использовать во втором табличном выражении для объединения таблицы самой с собой со сдвигом для формирования новой таблицы, в которой одна сессия будет записана в одну строку с колонками начала и окончания сессии. В такой таблице появляется возможность расчитать длину сессии как разницу между этими колонками. На третьем этапе группируем полученную объединенную таблицу по пользователям и с помощью агрегатной функции расчитываем среднее время сессии по поллученным интервалам в секундах.

```sql
WITH ordered_actions AS -- Таблица открытия и закрытия сессий пользователя, упорядоченных по времени
(
    SELECT user_id, timestamp, action,
        row_number () OVER (PARTITION BY user_id ORDER BY timestamp) AS seg_num
    FROM facebook_web_log
    WHERE action = 'page_load' OR action = 'page_exit'
),
matched_sessions AS -- Таблица сессий пользователя по одной на строку
(
    SELECT
        tab1.user_id,
        tab1.timestamp AS load_time, -- Время начала сессии
        tab2.timestamp AS exit_time, -- Время окончания сессии
        tab2.timestamp - tab1.timestamp AS session_duration -- Продолжительность сессии в секундах
    FROM ordered_actions AS tab1
    JOIN ordered_actions AS tab2 ON tab1.user_id = tab2.user_id AND
    tab1.seg_num = tab2.seg_num - 1
    WHERE tab1.action = 'page_load' AND tab2.action = 'page_exit' 
)
SELECT user_id, avg(session_duration) AS avg_session_duration
FROM matched_sessions
GROUP BY user_id;

```

 **Output**

|user_id|avg_session_duration|
|---|--:|
|0|1883.5|
|1|35|
</details>
<details>
<summary>Упражнение "Ранг активности": CTE, ранжирование с сортировкой по двум разным условиям <br>#WITH_AS #OVER #row_number()</br></summary>

ID 10351  

"Activity Rank"  
Find the email activity rank for each user. Email activity rank is defined by the total number of emails sent. The user with the highest number of emails sent will have a rank of 1, and so on. Output the user, total emails, and their activity rank. Order records by the total emails in descending order. Sort users with the same number of emails in alphabetical order.
In your rankings, return a unique value (i.e., a unique rank) even if multiple users have the same number of emails. For tie breaker use alphabetical order of the user usernames.

Table:  google_gmail_emails

id: int  
from_user: varchar  
to_user: varchar  
day: int   

**Solution**

В этом задании нужно проранжировать отправителей писем по их активности в исходяжих письмах, но результат вернуть такой, чтобы пользователи с одинаковым рангом (количеством писем) были дополнительно отсортированы по алфавиту. Для лучшей читаемости запроса сформируем таблицу пользователей с метрикой activity rank (total_emails) и потом будем ранжировать уже ее. Пронумеруем строки, как указано в задании, по убыванию количества написанных писем при поможи оконной функции row_number() с аргументами, сосотоящими из двух условий сортировки: по убыванию числа писем и по возрастанию имени пользователя по алфавиту.

```sql
WITH activity_rank AS -- Таблица рангов пользователей по e-mail активности
(
SELECT 
    from_user,
    count(from_user) AS total_emails
FROM google_gmail_emails
GROUP BY from_user
)
SELECT
    from_user,
    total_emails,
    row_number() OVER (ORDER BY total_emails DESC, from_user ASC) AS row_number -- Ранг с сортировккой по метрике и имени оправителя
FROM  activity_rank
LIMIT 5;

```

 **Output**


|from_user|total_emails|row_number|
|---|--:|--:|
|32ded68d89443e808|19|1|
|ef5fe98c6b9f313075|19|2|
|5b8754928306a18b68|18|3|
|55e60cfcc9dc49c17e|16|4|
|91f59516cb9dee1e88|16|5|
</details>
<details>
<summary>Упражнение "Количество повторных клиентов (Weekly active users - WAU)": оконная функция, self join <br>#lag() #OVER #JOIN</br></summary>

ID 10322  

"Finding User Purchases"  
Write a query that'll identify returning active users. A returning active user is a user that has made a second purchase within 7 days of any other of their purchases. Output a list of user_ids of these returning active users.

Table:  amazon_transactions

id: int  
user_id: int  
item: varchar  
created_at: datetime  
revenue: int  

**Solution**

Как обычно задача может быть решена несколькими способами в зависимости от того, как вычисляется длина окна совершения повторной покупки: [Query that'll identify returning active users in span of week.](https://stackoverflow.com/questions/65708991/query-thatll-identify-returning-active-users-in-span-of-week)

В нашем варианте для каждого пользователя в его окне рнасчитывается лаг между покупками в днях с помощью оконной функции lag(). Полученое количество дней совершения повторных покупок оборачивается в подзапрос из результатов которого можно отфильтровать активных в течение недели пользователей. Эту оконную функцию также можно упаковать в CTE и применять для расчета других метрик, например, TBP ― Time Between Purchases.

```sql
-- Вариант 1
--
SELECT DISTINCT (user_id) -- Уникальные активные пользователи
FROM
    (SELECT -- Пользователи с количеством дней повторных покупок
        user_id,
        created_at::date - lag(created_at::date) OVER (PARTITION BY user_id ORDER BY created_at) AS second_parchase_day
    FROM amazon_transactions) AS spd
WHERE second_parchase_day < 8 -- Пользователи с повторной покупкой не старше 7 дней
LIMIT 5;

```

Другим способом решения задачи может быть использование [self join](https://sky.pro/wiki/sql/osnovy-self-join-v-sql-ponyatie-i-realniy-primer-ispolzovaniya/) - соединения таблицы со своей копией с нужным условием.

```sql
-- Вариант 2
--
SELECT 
    DISTINCT (a.user_id) AS user_id
FROM amazon_transactions AS a
JOIN amazon_transactions AS b ON
    a.user_id = b.user_id 
    AND a.id <> b.id
    AND b.created_at::date - a.created_at::date BETWEEN 0 AND 7 
ORDER BY a.user_id
LIMIT 5;

```

 **Output**


|user_id|
|--:|
|100|
|103|
|105|
|109|
|110|
</details>
<details>
<summary>Упражнение "Проекты с перерасходом по ФОТ": объединение таблиц многие-ко-многим с условием фильтрации по формуле <br>#JOIN #date #ceil() #sum()</br></summary>

ID 10304  

"Risky Projects"  
Identify projects that are at risk for going overbudget. A project is considered to be overbudget if the cost of all employees assigned to the project is greater than the budget of the project.  

You'll need to prorate the cost of the employees to the duration of the project. For example, if the budget for a project that takes half a year to complete is $10K, then the total half-year salary of all employees assigned to the project should not exceed $10K. Salary is defined on a yearly basis, so be careful how to calculate salaries for the projects that last less or more than one year.  

Output a list of projects that are overbudget with their project name, project budget, and prorated total employee expense (rounded to the next dollar amount).  

HINT: to make it simpler, consider that all years have 365 days. You don't need to think about the leap years.  

Table:  linkedin_projects  

id: int  
title: varchar  
budget: int  
start_date: datetime  
end_date: datetime   

Table: linkedin_emp_projects  

emp_id: int  
project_id: int  

Table: inkedin_employees  

id: int  
first_name: varchar  
last_name: varchar  
salary: int  

**Solution**

Из исходных данных имеем три таблицы со связью многие-ко-многим через простые ключи. Объединяем таблицы через INNER JOIN чтобы получить зарплаты сотрудников с привязкой к проектам.  

Группируем проекты по названию и другим колонкам, участвующим в формуле расчета ФОТ. ФОТ проекта расчитываем как его продолжительность в днях, умноженная на сумму зарплат работников проекта за один день. В дальнейшем округляем ФОТ до целого функцией ceil().  

Теперь нужно отобрать проеты с перерасходом по ФОТ. Оборачиваем нашу объединенную таблицу в подзапрос и накладываем условие, что ФОТ больше бюджета проекта. Возвращенные строки и будут Risky Projects

```sql
SELECT 
    title, 
    budget, 
    ceil(prorated_expenses) AS prorated_employee_expense -- Округленный ФОТ проекта
FROM 
    (SELECT 
        title, 
        budget, 
        (end_date::date - start_date::date) * (sum(salary)/365) AS prorated_expenses -- Затраты по зарплате 
    FROM linkedin_projects AS lp 
    JOIN linkedin_emp_projects AS lep ON lp.id = lep.project_id 
    JOIN linkedin_employees AS le ON lep.emp_id = le.id 
    GROUP BY title, budget, end_date, start_date) AS t 
WHERE prorated_expenses > budget -- Перерасход проекта по зарплате
ORDER BY title ASC
LIMIT 5;

```
 **Output**

|title|budget|prorated_employee_expense|
|---|--:|--:|
|Project1|29498|36293|
|Project11|11705|31606|
|Project12|10468|62843|
|Project14|30014|36774|
|Project16|19922|21875|
</details>
<details>
<summary>Упражнение "Доля принятых запросов в друзья": общее табличное выражение (CTE), JOIN <br>#WITH_AS #LEFTJOIN</br></summary>

ID 10285  

"Acceptance Rate By Date"  
What is the overall friend acceptance rate by date? Your output should have the rate of acceptances by the date the request was sent. Order by the earliest date to latest.  

Assume that each friend request starts by a user sending (i.e., user_id_sender) a friend request to another user (i.e., user_id_receiver) that's logged in the table with action = 'sent'. If the request is accepted, the table logs action = 'accepted'. If the request is not accepted, no record of action = 'accepted' is logged.

Table: fb_friend_requests  

user_id_sender: varchar  
user_id_receiver: varchar  
date: datetime  
action: varchar  

**Solution**

Из общего потока событий необходимо вычленить события отправки запросов на добавление в друзья, принятия этих запросов другими пользователями и расчитать метрику Friend Acceptance Rate. Сгенерируем два общих табличных выражения с запросами на добавление в друзья и реакцией на них. Чтобы получить цепочку событий для каждого запроса объединим эти две таблицы по идентификаторам отправителей и откликнувшихся. На основе этой общей таблицы расчитаем метрику Friend Acceptance Rate.

```sql
-- Вариант 1
WITH sent_cte AS -- Запросы на добавление в друзья
(
SELECT 
  date, 
  user_id_sender,
  user_id_receiver
FROM fb_friend_requests
WHERE action = 'sent'
), 
accepted_cte AS -- Добавление в друзья
(
SELECT
    date, 
    user_id_sender,
    user_id_receiver
FROM fb_friend_requests
WHERE action = 'accepted'
)
SELECT 
    a.date,
    count(b.user_id_receiver)::numeric / count(a.user_id_sender)::numeric AS percentage_acceptance -- Friend Acceptance Rate
FROM sent_cte AS a
    LEFT JOIN accepted_cte AS b
    ON a.user_id_sender = b.user_id_sender 
    AND a.user_id_receiver = b.user_id_receiver
GROUP BY a.date;

```

Разбить одну колонку на две можно также селф джойном, но этот вариант имеет ограницения, поскольку в условиях объединения участваует время, которое имеет формат time, т.е. это дата без времени. Таким образом, если в датасете запрос на добавление в друзья и реакция на него произошли в один день, то в объединеную таблицу эта цепочка событий не войдет. Но в нашем датесете все события разнесены более, чем на один день и результат расчета метрики получается верным.

```sql
-- Вариант 2
--
SELECT 
    date, 
    count(accepted)::numeric / count(sent)::numeric AS percentage_acceptance
 FROM
    (SELECT 
        a.date AS date,
        a.action AS sent,
        b.action AS accepted 
    FROM fb_friend_requests AS a
    LEFT JOIN fb_friend_requests AS b
        ON a.user_id_sender = b.user_id_sender AND a.user_id_receiver = b.user_id_receiver
        AND a.date <> b.date 
    WHERE a.action = 'sent') AS t
GROUP BY date;

```

 **Output**

|date|percentage_acceptance|
|---|--:|
|2020-01-04|0.75|
|2020-01-06|0.667|
</details>
<details>
<summary>Упражнение "Ранжирование наиболее активных гостей в аккаунтах": ранжирование оконной функцией плотного ранга <br>#dense_rank() #OVER</br></summary>

ID 10159  

"Ranking Most Active Guests"  
Rank guests based on the total number of messages they've exchanged with any of the hosts. Guests with the same number of messages as other guests should have the same rank. Do not skip rankings if the preceding rankings are identical. Output the rank, guest id, and number of total messages they've sent. Order by the highest number of total messages first.  

Table: airbnb_contacts  

id_guest: varchar  
id_host: varchar  
id_listing: varchar  
ts_contact_at: datetime  
ts_reply_at: datetime  
ts_accepted_at: datetime  
ts_booking_at: datetime  
ds_checkin: datetime  
ds_checkout: datetime  
n_guests: int  
n_messages: int   

**Solution**

Группируем пользователей по id и расчитываем количество сообщений. Ранжируем пользователей по их суммарному кол-ву сообщений с помощью оконной функции не простого, а плотного ранга dense_rank(), поскольку по условию задачи одинаковым суммам сообщений нужно назначить одинаковый ранг без пропусков.

```sql
-- Вариант 1
SELECT
    dense_rank() OVER (ORDER BY sum(n_messages) DESC) AS ranking,
    id_guest,
    sum(n_messages) AS sum_n_messages
FROM airbnb_contacts
GROUP BY id_guest
LIMIT 5;

```

Первый шаг решения задачи по формированию таблицы с суммами сообщений пользователей можно обернуть в CTE или подзапрос

```sql
-- Вариант 2
--
SELECT
    dense_rank() OVER (ORDER BY sum_n_messages DESC) AS ranking,
    id_guest,
    sum_n_messages
FROM
    (SELECT 
        id_guest,
        sum(n_messages) AS sum_n_messages
    FROM airbnb_contacts
    GROUP BY id_guest) AS t
LIMIT 5;

```

 **Output**

|ranking|id_guest|sum_n_messages|
|--:|---|--:|
|1|882f3764-05cc-436a-b23b-93fea22ea847|20|
|1|62d09c95-c3d2-44e6-9081-a3485618227d|20|
|2|b8831610-31f2-4c58-8ada-63b3601ca476|17|
|2|91c2a883-04e3-4bbb-a7bb-620531318ab1|17|
|3|6133fb99-2391-4d4b-a077-bae40581f925|16|
</details>
<details>
<summary>Упражнение "Число арендованных номеров по национальности": объединение таблиц с условием <br>#JOIN #DISTINCT #count()</br></summary>

ID 10156  

"Number Of Units Per Nationality"  
Find the number of apartments per nationality that are owned by people under 30 years old.   
Output the nationality along with the number of apartments.  
Sort records by the apartments count in descending order.  

Table:  lairbnb_hosts   

host_id: int  
nationality: varchar  
gender: varchar  
age: int  

Table: airbnb_units  

host_id: int  
unit_id: varchar  
unit_type: varchar  
n_beds: int  
n_bedrooms: int  
country: varchar  
city: varchar  

**Solution**

Нациолнальность арендаторов и тип жилья с его характеристиками находятся в разных таблицах, которые нужно объединить. Используем внутреннее соединение с наложением условий по возрасту и типу жилья по условию задачи. Т.к. в одном гостиничном номере могут проживать несколоько человек и эти кортежи попадают в общую таблицу, то через DISTINCT подсчитываем только неповторяющееся жилье.

```sql
SELECT 
    nationality,
    count(DISTINCT unit_id) as apartment_count
FROM airbnb_hosts AS ah
JOIN airbnb_units AS au ON 
    ah.host_id = au.host_id
    WHERE ah.age < 30 AND au.unit_type = 'Apartment'
GROUP BY nationality
ORDER BY apartment_count DESC;

```
 **Output**

|nationality|apartment_count|
|---|--:|
|USA|2|

</details>
<details>
<summary>Упражнение "Число пользователей продукции Apple": объединение таблиц, условное выражение, ругулярные выражения POSIX <br>#JOIN #CASE #DISTINCT #count() #POSIX</br></summary>

ID 10141  

"Apple Product Counts"  
Find the number of Apple product users and the number of total users with a device and group the counts by language. Assume Apple products are only MacBook-Pro, iPhone 5s, and iPad-air. Output the language along with the total number of Apple users and users with any device. Order your results based on the number of total users in descending order.  

Table:  playbook_events   

user_id: int  
occurred_at: datetime  
event_type: varchar  
event_name: varchar  
location: varchar  
device: varchar   

Table: playbook_users  

user_id: int  
created_at: datetime  
company_id: int  
language: varchar  
activated_at: datetime  
state: varchar    

**Solution**

Названия продуктов и язык пользователей находятся в разных таблицах и их нужно объединить. Уникальных пользователей нужно сгруппировать по языку и при этом отфильтровать из общего списка только пользователей Apple  м дополнительно сравнить их статистику с пулом клиентов в целом. 

Вариант 1  
В общем табличнов выражении (CTE) генерруются две таблицы для двух кат егорий пользователей, которые потом объединяются в результирующую по коючевому полю language. Пользователи Apple фильтруются по шаблону с помощьюю регулярных выражений POSIX. Плюс этого подхода в том, что запрос можно отлаживать по частям и потом обернуть их в CTE.

```sql
WITH ntu AS ( -- Общее кол-во пользователей
    SELECT language, count(DISTINCT(pe.user_id)) AS n_total_users
        FROM playbook_events AS pe
        JOIN playbook_users AS pu ON pe.user_id = pu.user_id
    GROUP BY language
), nau AS ( --Кол-во пользователей продукции Aple
    SELECT language, count(DISTINCT(pe.user_id)) AS n_apple_users
        FROM playbook_events AS pe
        JOIN playbook_users AS pu ON pe.user_id = pu.user_id
    WHERE pe.device ~* 'MacBook.Pro' OR pe.device ~* 'iPhone.5s' OR pe.device ~* 'iPad.air'
    GROUP BY language)
SELECT ntu.language, COALESCE(n_apple_users, 0) AS n_apple_users, n_total_users
    FROM ntu
    LEFT JOIN nau ON ntu.language = nau.language
ORDER BY n_total_users DESC;

```
Вариант 2  
Делаем один общий JOIN, но чтобы получить сравнительную статистику, с помощью условного выражения и регулярных выражений  POSIX отделяем пользователей  Apple от всех клиентов в целом.  

```sql
SELECT users.language,
       COUNT(DISTINCT CASE
                           WHEN device ~* 'MacBook.Pro' THEN users.user_id
                           WHEN device ~* 'iPhone.5s' THEN users.user_id
                           WHEN device ~* 'iPad.air' THEN users.user_id
                           ELSE NULL
                       END) AS n_apple_users,
             COUNT(DISTINCT users.user_id) AS n_total_users
FROM playbook_users AS users
JOIN playbook_events AS events ON users.user_id = events.user_id
GROUP BY users.language
ORDER BY n_total_users DESC;

```
Если есть уверенность в том, что данные хорошего качества, то можно обойтись и без проверки строк регулярными выражениями POSIX.  

```sql
SELECT users.language,
       COUNT(DISTINCT CASE
                           WHEN device IN ('macbook pro',
                                           'iphone 5s',
                                           'ipad air') THEN users.user_id
                           ELSE NULL
                       END) AS n_apple_users,
             COUNT(DISTINCT users.user_id) AS n_total_users
FROM playbook_users AS users
JOIN playbook_events AS events ON users.user_id = events.user_id
GROUP BY users.language
ORDER BY n_total_users DESC;

```
 **Output**

|language|n_apple_users|n_total_users|
|---|--:|--:|
|english|11|45|
|spanish|3|9|
|japanese|2|6|
|french|0|5|
|russian|0|5|
|chinese|1|4|
|german|1|3|
|portugese|1|3|
|indian|0|2|
|arabic|0|2|
|italian|1|1|

</details>
<details>
<summary>Упражнение "Спам-посты": подсчет процентов с объединение таблиц и условным выражением <br>#CASE #JOIN #count() #sum()</br> </summary>

ID 10134  

"Spam Posts"  
Calculate the percentage of spam posts in all viewed posts by day. A post is considered a spam if a string "spam" is inside keywords of the post. Note that the facebook_posts table stores all posts posted by users. The facebook_post_views table is an action table denoting if a user has viewed a post. 

Table:  facebook_posts  

post_id: int  
poster: int  
post_text: varchar  
post_keywords: varchar  
post_date: datetime    

Table: facebook_post_views  

post_id: int  
viewer_id: int    

**Solution**

Таблица facebook_post_views представляет собой идентификаторы просмотренных постов и т.к. только они нам и нужны, то джойним к ней в данном случае справа таблицу facebook_posts с описанием самих постов. Для подсчета процентов в числителе расчитываем сумму "промаркированных" единичкой с помощью условного выражения постов, которые обозначены как spam. В качестве знаменателя формулы берем количество идентификаторов постов, т.к. там точно не будет пустых значений. 

```sql
SELECT 
    post_date,
    sum(CASE WHEN post_keywords ~ '#spam#' IS TRUE THEN 1 ELSE 0 END)::NUMERIC(4,1) / count(fpv.post_id) * 100 AS spam_share
FROM facebook_posts AS fp
RIGHT JOIN facebook_post_views AS fpv ON fp.post_id = fpv.post_id
GROUP BY post_date;

```

Stratascratch  предлагает другое решение, основанное на INNER JOIN. Это внутренне соединение делается дважды. Одно, чтобы отфильтровать из общей объединенной таблицы прочитанных и непрочитанных постов общую сумму прочитанных постов. Другое - чтобы найти сумму прочитанных постов, помеченных как spam. Эти две таблицы джойнятся, чтобы получить колонки для расчета процеентов.   

```sql
SELECT spam_summary.post_date,
       (n_spam/n_posts::float)*100 AS spam_share
FROM
  (SELECT post_date,
          sum(CASE
                  WHEN v.viewer_id IS NOT NULL THEN 1
                  ELSE 0
              END) AS n_posts
   FROM facebook_posts p
   JOIN facebook_post_views v ON p.post_id = v.post_id
   GROUP BY post_date) posts_summary
LEFT JOIN
  (SELECT post_date,
          sum(CASE
                  WHEN v.viewer_id IS NOT NULL THEN 1
                  ELSE 0
              END) AS n_spam
   FROM facebook_posts p
   JOIN facebook_post_views v ON p.post_id = v.post_id
   WHERE post_keywords ilike '%spam%'
   GROUP BY post_date) spam_summary ON spam_summary.post_date = posts_summary.post_date;

```

 **Output**

|post_date|spam_share|
|---|---|
|2019-01-02|50|
|2019-01-01|100|
</details>
<details>
<summary>Упражнение "Процент высланных по почте заказов": подсчет процентов с объединением таблиц и CTE c условным выражением для строковых данных <br>#CTE #NULLIF #CASE #JOIN #count() #sum()</br></summary>

ID 10090  

Find the percentage of shipable orders.  
Consider an order is shipable if the customer's address is known. 

Table:  orders  

id: int  
cust_id: int  
order_date: datetime  
order_details: varchar  
total_order_cost: int  

Table: customers  

id: int  
first_name: varchar  
last_name: varchar  
city: varchar  
address: varchar  
phone_number: varchar    

**Solution**

Сначала объединим две таблицы, чтобы сопосьтавить каждому заказу адрес доставки. Для удобства завернем все в CTE, где с помощью условных выражений промаркируем пустые адреса для доставки, которые найдем с помощью NULLIF. Эту колонку используем в формуле для расчета процентов.  

```sql
WITH t AS ( -- Адреса сделанных заказов
SELECT
CASE WHEN NULLIF(address, '') IS NULL THEN 0 -- Если в адресе NULL или пустая строка, то 0
    ELSE 1
END AS empty_addr_flag -- флаг пустых строк в адресах
FROM orders AS o
JOIN customers AS c ON o.cust_id = c.id
)
SELECT
sum(empty_addr_flag)::numeric / count(empty_addr_flag)::numeric * 100 AS percent_shipable
FROM t; 

```

Stratascratch  предлагает несколько иное решение с использованием подзапросов без CTE. Но тогда дважды вычисляется условное выражение: один раз для маркировки статуса адреса булевым значением и второй раз для конвертации этих значений в числа для их подсчета в формуле процента.    

```sql
SELECT
    100 * SUM(CASE WHEN is_shipable THEN 1 ELSE 0 END) :: NUMERIC / COUNT(*) AS percent_shipable
FROM
    (SELECT
        o.id,
        CASE WHEN address IS NULL THEN False ELSE True END AS is_shipable
    FROM orders o
    INNER JOIN customers c ON o.cust_id = c.id) base;

```

 **Output**

|percent_shipable|
|:--|
|28|
</details>
<details>
<summary>Упражнение "Поиск совпадающих пар host - guest по соцдему пользователей ": объединение таблиц и фильтрация уникальных пар значений <br>#JOIN #DISTINCT</br></summary>

ID 10078 

Find matching hosts and guests pairs in a way that they are both of the same gender and nationality.  
Output the host id and the guest id of matched pair.

Table:  airbnb_hosts  

host_id: int  
nationality: varchar  
gender: varchar  
age: int   

Table: airbnb_guests 

guest_id: int  
nationality: varchar  
gender: varchar  
age: int      

**Solution**

Объединяем две таблицы по совпадающим парам соцдема национальность - пол. Возвращаем уникальные строки по колонкам  host - guest.

```sql
SELECT DISTINCT host_id, guest_id
FROM airbnb_hosts AS ah
JOIN airbnb_guests AS ag ON 
    ah.nationality = ag.nationality AND ah.gender = ag.gender;

```

 **Output**

|host_id|guest_id|
|---|---|
|0|9|
|1|5|
|2|1|
|3|7|
|4|0|
|5|2|
|6|4|
|7|10|
|8|3|
|9|8|
|10|6|
|11|11|
</details>
<details>
<summary>Упражнение "Итоговый доход работников по должностям и полу": Общее табличное выражение с бонусами объединяется с таблицей работников с группировкой по должностям и полу<br>#CTE #WITH AS #JOIN #GROUP BY #avg()</br></summary>

ID 10077

Income By Title and Gender.  
Find the average total compensation based on employee titles and gender. Total compensation is calculated by adding both the salary and bonus of each employee. However, not every employee receives a bonus so disregard employees without bonuses in your calculation. Employee can receive more than one bonus. Output the employee title, gender (i.e., sex), along with the average total compensation.

Table:  sf_employee 

id: int  
first_name: varchar  
last_name: varchar  
age: int  
sex: varchar  
employee_title: varchar  
department: varchar  
salary: int  
target: int  
email: varchar  
city: varchar  
address: varchar  
manager_id: int    

Table: sf_bonus 

worker_ref_id: int  
bonus: int      

**Solution**

По условию задачи одни работник может получать несколько бонусов. Чтобы правильно посчитать среднюю итоговую зарплату по должностям и полу, сгенерируем в CTE таблицу работниок с суммарным количеством бонусов и уже ее будем объединять с основной таблицей чере INNER JOIN, чтобы отсечь работников без бонусов. Полученную таблицу группируем по должности и полу и расчитываем агрегатную функцию среднего по сумме зарплат и бонусо внутри сформированных групп. Вместо CTЕ можно было бы использовать подзапрос.

```sql
WITH total_sf_bonus AS ( -- Суммарные бонусы работника
    SELECT worker_ref_id, sum(bonus) AS bonus 
    FROM sf_bonus GROUP BY worker_ref_id
    )
SELECT 
    employee_title, 
    sex, 
    avg(salary + bonus) AS avg_compensation
FROM sf_employee AS se
JOIN total_sf_bonus AS tsb 
    ON se.id = tsb.worker_ref_id
GROUP BY employee_title, sex;

```

 **Output**

employee_title|sex|avg_compensation|
|:--|:--|---|
|Senior Sales|M|5350|
|Auditor|M|2200|
|Manager|F|209500|
|Sales|M|4600|
</details>
<details>
<summary>Упражнение "Самое высокое потребление энергии": объединение множеств, подзапросы + нетипичное применение джойна<br>#UNION #sum() #dense_rank() #OVER #CTE #WITH #max() #JOIN</br></summary>

ID 10064 

Highest Energy Consumption  
Find the date with the highest total energy consumption from the Meta/Facebook data centers. Output the date along with the total energy consumption across all data centers.  

Table: fb_eu_energy   

date: datetime  
consumption: int   

Table: fb_asia_energy   

date: datetime  
consumption: int   

Table: fb_na_energy  

date: datetime  
consumption: int   

**Solution**

Используем вложенные подзапросы. Сначала объединяем три множества (три таблицы) чере UNION. Затем группируем общее множество по дате и агрегатной функцией расчитываем потребление энергии внутри дат. Наконец ранжируем потребление оконной функцией плотного ранга dense_rank() по потреблению энергии и оборачиваем все в следующий подзапрос, чтобы не потерять даты с одинаковым максимальным потреблением. Отфильтровываем ранг = 1 и получаем решение задачи и как и предполагалось, есть две разные даты с одинаковым максимальным потреблением энергии. 

```sql
SELECT date, total_energy
FROM
    (SELECT date, 
        total_energy,
        dense_rank() OVER (ORDER BY total_energy DESC) AS total_energy_rank
    FROM
        (SELECT date, sum(consumption) AS total_energy
            FROM
                (SELECT date, consumption FROM fb_eu_energy
                UNION ALL
                SELECT date, consumption FROM fb_asia_energy
                UNION ALL
                SELECT date, consumption FROM fb_na_energy) AS energy_consumption
        GROUP BY date) AS energy_cons_by_date) AS ranked_consumption
WHERE total_energy_rank = 1;

```
StrataScrath предлагает несколько иное и довольно неожиданное решение. Вместо подзапросов используются три CTE, где одно ображается к другому. Но как возвратить максимум потребления, если их несколько одинаковых, а агрегатная функция max() возвращает только одно число и оно будет выбрано случайным образом при наличии одинаковых? Мы использовали оконную функцию плотного ранга dense_rank(), но для данной задачи есть и другое решение с помощью INNER JOIN. Для этого и нужны CTE. Таблицу со сгруппированными по датам данными потребления джойним с производной CTE-таблицей с единственной строкой максимального потребления. Если в основной таблице только один максимум по потреблению, то он и возвращается внутренним джойном, а если их несколько, то они оказываются в разных строчках финальной объединенной таблицы.

```sql
WITH total_energy AS
  (SELECT * FROM fb_eu_energy eu
   UNION ALL 
   SELECT * FROM fb_asia_energy asia
   UNION ALL 
   SELECT * FROM fb_na_energy na),
--
energy_by_date AS
  (SELECT date, sum(consumption) AS total_energy
   FROM total_energy
   GROUP BY date
   ORDER BY date ASC),
--
max_energy AS
  (SELECT max(total_energy) AS max_energy
   FROM energy_by_date)
--
SELECT ebd.date, ebd.total_energy
FROM energy_by_date ebd
JOIN max_energy me ON ebd.total_energy = me.max_energy;

```
 **Output**

|date|total_energy|
|---|---|
|2020-01-06|1250|
|2020-01-07|1250|
</details>

## SQL-задачи из других источников
<details>
<summary>Подбор туристических поездок по определенным параметрам  
<br>#CREATE DATABASE #CREATE TABLE #INSERT INTO #JOIN #dense_rank() #OVER</br></summary>
	
Определите, для какой местности в БД есть наибольшее количество предложений туров,  не дороже 80000 за 7 дней и более и с водоемом.  
В ответ запишите количество предложений для этой местности, отвечающее заданным условиям. 

 **Solution**
 Сщздаем базу данных со связанными по ключам таблицами и заполняем их данными.  
 
```sql
CREATE DATABASE tourism;

CREATE TABLE locations -- Расположение курорта
(id integer PRIMARY KEY, -- Ключ расположения курорта
location varchar(9)	); -- Месторасположение курорта

INSERT INTO locations (id, location) VALUES
(1, 'Sakhalin'),
(2, 'Altai'),
(3, 'Baikal'),
(4, 'Siberia'),
(5, 'Karelia');

CREATE TABLE recriations -- Курорт
(id integer PRIMARY KEY, -- Ключ курорта
recriation_name varchar(10), -- Наименование курорта
location_id integer REFERENCES locations, -- Ключ расположения курорта
reservoir integer); -- Наличие водоема (1 - да, 0 - нет)

INSERT INTO recriations (id, recriation_name, location_id, reservoir) VALUES 
(1, 'Cuffs', 2, 1),
(2, 'Mill', 3, 1),
(3, 'Chermal', 4, 0),
(4, 'Albatros', 1, 1),
(5, 'Hamar', 3, 1),
(6, 'Sheregesh', 4, 1),
(7, 'Viking', 2, 1),
(8, 'Santa', 5, 1),
(9, 'Bear Creek', 1, 0),
(10, 'Barrel', 4, 1),
(11, 'Kaya', 2, 1),
(12, 'Alatan', 2, 1);

CREATE TABLE travel_offers -- Туры
(id integer PRIMARY KEY, -- Ключ тура
recriation_id integer REFERENCES recriations, -- Ключ курорта
price numeric(6), -- Цена
duration integer, -- Продолжительность
travel_month varchar(9)); -- Месяц

INSERT INTO travel_offers (id, recriation_id, price, duration, travel_month) VALUES 
(1, 5, 70000, 7, 'August'),
(2, 2, 49000, 6, 'July'),
(3, 12, 140000, 14, 'August'),
(4, 10, 70500, 7, 'August'),
(5, 1, 50000, 7, 'August'),
(6, 3, 100000, 12, 'September'),
(7, 4, 60000, 6, 'August'),
(8, 4, 99000, 10, 'August'),
(9, 6, 71000, 7, 'May'),
(10, 9, 68000, 7, 'August'),
(11, 5, 168000, 12, 'June'),
(12, 7, 79000, 8, 'August'),
(13, 11, 142000, 14, 'August'),
(14, 9, 72000, 12, 'August'),
(15, 11, 50000, 6, 'August');

```
На первом этапе джойним таблицы сразу отфильтровывая нужные нам туры по параметрам цены, продолжительности и наличию водоема.  
Затем строим функцию плотного ранга для ранжирования туров и подсчитываем количество предложений по этим турам.  
Оборачивам все это в подзапрос, чтобы отобрать туры с рангом = 1.  

```sql
SELECT location, proposals_num
FROM 
	(SELECT 
		location,
		count(location) AS proposals_num,
		dense_rank() OVER (ORDER BY count(location) DESC) AS proposal_rank
	FROM travel_offers AS tro 
	JOIN recriations AS r ON tro.recriation_id = r.id
		AND tro.price <= 80000 
		AND tro.duration >= 7
		AND r.reservoir = 1
	JOIN locations AS l ON r.location_id = l.id
	GROUP BY location) AS t -- Проранжированная таблица
WHERE proposal_rank = 1; -- Отбираем строки с рангом = 1

```
Удаляем базу данных.

```sql
DROP DATABASE tourism;

```

 **Output**
 
|location|proposals_num|
|---|--:
|Altai|2|
|Siberia|2|
</details>
<details>
<summary>В каждой категории материала найти id материала с минимальным приоритетом использования.  
<br>#CREATE TEMP TABLE #INSERT INTO #JOIN #dense_rank() #OVER</br></summary>
	
По таблице (Код_материала, ИД, Приоритет) написать запрос, который для каждого кода материала возвращает ИД, у которого минимальный приоритет.

 **Solution**
 Создаем временную таблицу и заполняем ее данными.  
 
```sql
CREATE TEMP TABLE material (
	code integer,
	id integer,
	priority integer,
	PRIMARY KEY (id)
);

INSERT INTO material (code, id, priority) VALUES
(100, 1, 3),
(100, 2, 1),
(100, 3, 2),
(103, 4, 1),
(103, 5, 1),
(103, 6, 2),
(102, 7, 1),
(102, 8, 2),
(102, 9, 3),
(111, 10, 3),
(111, 11, 2),
(111, 12, 1),
(112, 13, 2),
(112, 14, 3),
(112, 15, 3);

```
Сортировку по приоритету материала внутри каждой категории типа материала решаем за счет оконной функции  плотного ранга dense_rank() с сортировкой по колонке Приоритет по возрастанию.  Чтобы найти минимальный приоритет оборачиваем все в подзапрос и фильтруем ранг = 1. Используем функцию плотного ранга вместо обычного, чтобы не потерять материалы, которые могут иметь одинаковый приоритет, что и произошло с материалами с id = 5 и 4.

```sql
SELECT *
FROM (
	SELECT code, priority, id AS d, dense_rank() OVER (PARTITION BY code ORDER BY priority) AS r
	FROM material
	) AS t
WHERE r = 1;

```
 **Output**
 
```sql
material_code|priority|id|m_rank|
-------------+--------+--+------+
          100|       1| 2|     1|
          102|       1| 7|     1|
          103|       1| 5|     1|
          103|       1| 4|     1|
          111|       1|12|     1|
          112|       2|13|     1|

6 row(s) fetched.

```
</details>

