These are some of the queries I used to answer to the business inquieries for this project. 

For answering to what were the sales per month, I used: 
```
SELECT EXTRACT (MONTH FROM payment_date) AS month_number, 						
CASE WHEN (EXTRACT (MONTH FROM payment_date) = 2) THEN 'February'						
WHEN (EXTRACT (MONTH FROM payment_date) = 3) THEN 'March'						
WHEN (EXTRACT (MONTH FROM payment_date) = 4) THEN 'April'					
ELSE 'May' END AS month_name,					
SUM(amount)						
FROM payment						
GROUP BY month_number						
ORDER BY month_number;						
```
To see which categories contributed to most of the revenue: 
```
SELECT c.name, ROUND(SUM(p.amount),0) AS revenue					
FROM category c					
JOIN film_category fc ON c.category_id = fc.category_id					
JOIN film f ON fc.film_id = f.film_id					
JOIN inventory i ON f.film_id = i.film_id					
JOIN rental r ON i.inventory_id = r.inventory_id					
JOIN payment p ON r.rental_id = p.rental_id					
GROUP BY c.name					
ORDER BY SUM(p.amount) DESC					
```
To explore the paying history of each customer, I wanted to see the transactions per month and their total sum.
For this query I used CTEs and joins. 
```
SELECT *, (COALESCE(total_sales_february, 0) 					
+(COALESCE(total_sales_march, 0) 					
+(COALESCE(total_sales_april, 0) 					
+ COALESCE(total_sales_may, 0)) AS total_sales					
FROM 					
(	WITH paying_customers_cte				
(unique_customer_id) AS					
(SELECT DISTINCT customer_id AS unique_customer_id					
FROM payment),					
					
februarytransactions_cte					
(customer_id_february, 					
 sales_february) AS					
(SELECT customer_id, 					
SUM(amount)AS sales_february					
FROM payment					
WHERE payment_date < '2007-03-01'					
GROUP BY customer_id					
ORDER BY sales_february DESC),					
					
marchtransactions_cte					
(customer_id_march, 					
 sales_march) AS					
(SELECT customer_id, 					
SUM(amount)AS sales_march					
FROM payment					
WHERE payment_date BETWEEN '2007-03-01' AND '2007-03-31'					
GROUP BY customer_id					
ORDER BY sales_march DESC),					
					
apriltransactions_cte					
(customer_id_april, 					
 sales_april) AS					
(SELECT customer_id, 					
SUM(amount)AS sales_april					
FROM payment					
WHERE payment_date BETWEEN '2007-04-01' AND '2007-04-30'					
GROUP BY customer_id					
ORDER BY sales_april DESC),					
					
maytransactions_cte					
(customer_id_may, 					
 sales_may) AS					
(SELECT customer_id, 					
SUM(amount)AS sales_may					
FROM payment					
WHERE payment_date > '2007-04-30'					
GROUP BY customer_id					
ORDER BY sales_may DESC)					
					
SELECT unique_customer_id 					
,SUM(sales_february)AS total_sales_february					
,SUM(sales_march)AS total_sales_march					
,SUM(sales_april)AS total_sales_april					
,SUM(sales_may)AS total_sales_may					
FROM paying_customers_cte					
LEFT JOIN februarytransactions_cte					
ON paying_customers_cte.unique_customer_id =					
februarytransactions_cte.customer_id_february					
LEFT JOIN marchtransactions_cte					
ON paying_customers_cte.unique_customer_id =					
marchtransactions_cte.customer_id_march					
LEFT JOIN apriltransactions_cte					
ON paying_customers_cte.unique_customer_id =					
apriltransactions_cte.customer_id_april					
LEFT JOIN maytransactions_cte					
ON paying_customers_cte.unique_customer_id =					
maytransactions_cte.customer_id_may					
GROUP BY unique_customer_id					
ORDER BY total_sales_february DESC NULLS LAST					
, total_sales_march DESC					
, total_sales_april DESC					
, total_sales_may DESC					
					
) a					
					
ORDER BY total_sales DESC;					
```
