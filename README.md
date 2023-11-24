# Hello there! 
## This is my 2nd SQL query project, thank you for your interest!
## In this project i wanted to get a few details that i thought may be useful for business needs, like knowing their selling performence and their customers.

##  dataset structure:

<img width="472" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/c3b7b26a-3c83-4346-bc63-f2b8fa021ed8">


# Details i wanted to get:

### 1) how many orders per item?
### 2) top selling items in terms of value
### 3) gender details
### 4) sales through time
### 5) top 10 selling items on BEST season
### 6) top 10 selling items for every gender
### 7) orders distribution by week days (and differences between males and females)
### 8) top selling items by value on every day
### 9) customers value and change from average
### 10) is there any weekly seasonality for each order size? (micro,small, medium, big, mega)
### 11) how many orders per day?


## 1)  how many orders per item?
```ruby
SELECT i.item_name,
SUM(o.order_quantity) as quantity_sold
FROM items i
LEFT JOIN orders o
ON i.item_id = o.item_id
GROUP BY i.item_name
HAVING SUM(o.order_quantity) > 0
ORDER BY quantity_sold DESC
```
<img width="350" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/4b4e232c-f9ca-487a-8014-36044e1e6641">

## 2) top selling items in terms of value
```ruby
SELECT i.item_name,
SUM(o.order_quantity*i.item_price) AS value_
FROM items i
LEFT JOIN orders o
ON i.item_id = o.item_id
GROUP BY i.item_name
HAVING SUM(o.order_quantity*i.item_price) > 0
ORDER BY value_ DESC
```
<img width="394" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/f01a8bee-15b0-4991-a51b-006d1c1270d4">

## 3)  gender details
```ruby
with gends as (
SELECT gender,
		SUM(CASE WHEN gender = 'male' THEN (order_quantity*i.item_price) ELSE (order_quantity*i.item_price) END) AS total_value,
		COUNT(CASE WHEN gender = 'male'  THEN (order_quantity) ELSE order_quantity END) AS total_count
FROM customers c JOIN orders o
ON c.id = o.customer_id
JOIN items i 
ON i.item_id = o.item_id
GROUP BY gender
)

SELECT gender,
		FLOOR(total_value) AS total_value,
				total_count,
				FLOOR(total_value / total_count) AS avg_order_value 
FROM gends
```		
### -- we can see that in total, women order more and worth more in terms of value, but the average order value is higher in men.

<img width="301" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/a8dde05d-3cb7-4660-9762-f80ebe704f0b">

<img width="304" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/0cf2d582-adba-4ad8-85ce-bb3ff5633e95">

<img width="303" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/bdf18be6-64fa-4d63-a2e6-f14fbc447bee">


## 4) sales through time, sum by months
```ruby
with seasons as (
SELECT CONCAT(MONTH(order_date), '/', YEAR(order_date)) AS new_date,
o.order_quantity*i.item_price AS order_value,
CASE	
	WHEN
	MONTH(o.order_date) BETWEEN 03 AND 05 THEN 'Spring'
		WHEN
			MONTH(o.order_date) BETWEEN 06 AND 08 THEN 'Summer'
			WHEN
				MONTH(o.order_date) BETWEEN 09 AND 11 THEN 'Autumn'
		ELSE 'Winter' 
		END as Season
FROM orders o
JOIN 
items i
ON i.item_id = o.item_id 
)
```
### -- to see orders through time:
```ruby
SELECT season, new_date,
SUM(order_value) AS season_sales,
RANK () OVER (ORDER BY MIN(new_date) ASC) AS date_rank
FROM seasons
GROUP BY  season, new_date
ORDER BY MIN(new_date) 
```
<img width="306" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/57116745-28cf-4ba3-86cc-93625df7413d">

## 5) top selling items in the BEST season

### -- to find the best season (its winter, probably because of Christmas):
```ruby
with seasons as (
SELECT CONCAT(MONTH(order_date), '/', YEAR(order_date)) AS new_date,
o.order_quantity*i.item_price AS order_value,
CASE	
	WHEN
	MONTH(o.order_date) BETWEEN 03 AND 05 THEN 'Spring'
		WHEN
			MONTH(o.order_date) BETWEEN 06 AND 08 THEN 'Summer'
			WHEN
				MONTH(o.order_date) BETWEEN 09 AND 11 THEN 'Autumn'
		ELSE 'Winter' 
		END as Season
FROM orders o
JOIN 
items i
ON i.item_id = o.item_id 
) ```

SELECT season,
SUM(order_value) AS season_sales
FROM seasons
GROUP BY  season
ORDER BY season_sales DESC
```
<img width="303" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/98c72f58-2777-41c7-be71-f9794df88164">


### -- top 10 selling items on Winter (months 9,10,11):
```ruby
SELECT TOP 10 i.item_name,
SUM(o.order_quantity) AS units_sold,
i.item_price
FROM orders o 
	JOIN 
	items i		
		ON i.item_id = o.item_id
WHERE MONTH(o.order_date) BETWEEN '09' and '11'
GROUP BY i.item_name, i.item_price
ORDER BY units_sold DESC
```
<img width="246" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/ff17dd6b-4e69-474c-8879-ccbeedd991c9">

## 6) top 10 selling items for every gender:
```ruby
with ranked_items as (
    SELECT
 i.item_name,
SUM(o.order_quantity) AS units_sold,
i.item_price,
c.gender,
ROW_NUMBER() OVER (PARTITION BY c.gender ORDER BY SUM(o.order_quantity) DESC) AS rnk
FROM
    orders o
		 JOIN
			 items i ON i.item_id = o.item_id
			 JOIN
				customers c ON c.id = o.customer_id

GROUP BY i.item_name, i.item_price, c.gender
)
```
```ruby
SELECT
item_name,
units_sold,
item_price,
gender
FROM
	ranked_items
WHERE  rnk <= 10
```
<img width="283" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/c51c4f61-b4fe-41cc-83fd-4b2e5f48b5ae">

## 7) orders distribution by week days (and differences between males and females)
```ruby
 SELECT COUNT( DATENAME(WEEKDAY, order_Date)) as orders_count,
 DATENAME(WEEKDAY, order_Date) as week_day
FROM orders 
GROUP BY  DATENAME(WEEKDAY, order_Date) 
order by orders_count DESC
```
<img width="330" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/afaf5ecd-c436-44bd-89c6-6a77867f5128">


### -- orders count difference between males and females
```ruby
 SELECT COUNT( DATENAME(WEEKDAY, order_Date)) as orders_count,
 DATENAME(WEEKDAY, order_Date) as week_day
FROM orders o JOIN customers c
ON c.id = o.customer_id
WHERE gender = 'male'
GROUP BY  DATENAME(WEEKDAY, order_Date) 
order by orders_count DESC
```
<img width="307" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/dfdaeaa0-4130-4e2e-8a53-e862f06b064a">

```ruby
 SELECT COUNT( DATENAME(WEEKDAY, order_Date)) as orders_count,
 DATENAME(WEEKDAY, order_Date) as week_day
FROM orders o JOIN customers c
ON c.id = o.customer_id
WHERE gender = 'female'
GROUP BY  DATENAME(WEEKDAY, order_Date) 
order by orders_count DESC
```
<img width="306" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/136ef730-ac49-4708-b0d3-0967c405158c">

### -- tuesday is the busiest.

## 8) top selling items by value on every day
--- top 10 selling items by value on every day
```ruby
SELECT TOP 10 i.item_name,
SUM(o.order_quantity*i.item_price) AS value_
FROM items i
LEFT JOIN orders o
ON i.item_id = o.item_id
WHERE DATENAME(weekday, o.order_date) = 'sunday'
GROUP BY i.item_name
HAVING SUM(o.order_quantity*i.item_price) > 0
ORDER BY value_ DESC
```
<img width="196" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/10fef0e1-28df-4074-b179-24d66ee28ad7">

```ruby
SELECT TOP 10 i.item_name,
SUM(o.order_quantity*i.item_price) AS value_
FROM items i
LEFT JOIN orders o
ON i.item_id = o.item_id
WHERE DATENAME(weekday, o.order_date) = 'monday'
GROUP BY i.item_name
HAVING SUM(o.order_quantity*i.item_price) > 0
ORDER BY value_ DESC
```
<img width="181" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/b48777a5-4acc-46ac-8876-2255028f7eb8">

```ruby
SELECT TOP 10 i.item_name,
SUM(o.order_quantity*i.item_price) AS value_
FROM items i
LEFT JOIN orders o
ON i.item_id = o.item_id
WHERE DATENAME(weekday, o.order_date) = 'tuesday'
GROUP BY i.item_name
HAVING SUM(o.order_quantity*i.item_price) > 0
ORDER BY value_ DESC
```
<img width="194" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/be422ed2-ad7b-4677-b17d-16d110627f83">

```ruby
SELECT TOP 10 i.item_name,
SUM(o.order_quantity*i.item_price) AS value_
FROM items i
LEFT JOIN orders o
ON i.item_id = o.item_id
WHERE DATENAME(weekday, o.order_date) = 'wednesday'
GROUP BY i.item_name
HAVING SUM(o.order_quantity*i.item_price) > 0
ORDER BY value_ DESC
```
<img width="195" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/2675cbae-a2d0-468e-b5ce-3d8731ff8b25">

```ruby
SELECT TOP 10 i.item_name,
SUM(o.order_quantity*i.item_price) AS value_
FROM items i
LEFT JOIN orders o
ON i.item_id = o.item_id
WHERE DATENAME(weekday, o.order_date) = 'thursday'
GROUP BY i.item_name
HAVING SUM(o.order_quantity*i.item_price) > 0
ORDER BY value_ DESC
```
<img width="195" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/c530175e-80ee-42c5-b787-1febe7641b9f">

```ruby
SELECT TOP 10 i.item_name,
SUM(o.order_quantity*i.item_price) AS value_
FROM items i
LEFT JOIN orders o
ON i.item_id = o.item_id
WHERE DATENAME(weekday, o.order_date) = 'friday'
GROUP BY i.item_name
HAVING SUM(o.order_quantity*i.item_price) > 0
ORDER BY value_ DESC
```
<img width="195" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/af5f072c-fa4f-4763-b94a-6fcbcc72e947">

```ruby
SELECT TOP 10 i.item_name,
SUM(o.order_quantity*i.item_price) AS value_
FROM items i
LEFT JOIN orders o
ON i.item_id = o.item_id
WHERE DATENAME(weekday, o.order_date) = 'saturday'
GROUP BY i.item_name
HAVING SUM(o.order_quantity*i.item_price) > 0
ORDER BY value_ DESC
```
<img width="186" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/2d06bec7-658d-4815-8456-6920601c043b">


## 9) customers value
```ruby
SELECT
c.gender,
c.first_name + ' ' + c.last_name AS customer_name,
SUM(o.order_quantity * i.item_price) AS total_value,
SUM(o.order_quantity * i.item_price) - AVG(SUM(o.order_quantity * i.item_price)) OVER () AS change_from_avg

FROM
customers c

LEFT JOIN
orders o ON c.id = o.customer_id

JOIN
items i ON i.item_id = o.item_id

GROUP BY
c.first_name, c.last_name, c.gender

ORDER BY 
total_value DESC
```
<img width="251" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/2716445f-c708-4ad1-a842-989c3be4f269">


## 10) seasonality for each order size (micro,small, medium, big, mega)
```ruby
SELECT DATENAME(weekday, order_date) as day,
		SUM((CASE WHEN order_quantity < 11 THEN order_quantity END)) AS micro,
		SUM((CASE WHEN order_quantity BETWEEN 11 AND 20 THEN order_quantity END)) AS small,
		SUM((CASE WHEN order_quantity BETWEEN 21 AND 40 THEN order_quantity END)) AS 'medium',
		SUM((CASE WHEN order_quantity BETWEEN 41 AND 75 THEN order_quantity END)) AS big,
		SUM((CASE WHEN order_quantity BETWEEN 76 AND 100 THEN order_quantity END)) AS mega
FROM orders 
GROUP BY DATENAME(weekday, order_date)
```
<img width="256" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/e28ae518-b116-4743-9682-f9daf8a958db"> <img width="269" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/05feeabf-8368-4c36-b83c-179628d670bb">
<img width="267" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/6e2f3683-2254-4271-860d-f1e975332192"> <img width="267" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/c55f2f7c-7686-4f1d-b970-53736fbfc184">
<img width="270" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/1bd7d57c-55c9-459a-95a4-9a175ef55627">


## 11) sales per day (quantity)
```ruby
SELECT DATENAME(weekday, order_date) AS day,
SUM(order_quantity) AS units_sold
FROM orders 
GROUP BY DATENAME(weekday, order_date)
ORDER BY units_sold DESC
```

  <img width="287" alt="image" src="https://github.com/EranEid24/Orders-query/assets/149265837/1c3b59c8-568e-4b2f-8ecc-16d4eddd6b43">
