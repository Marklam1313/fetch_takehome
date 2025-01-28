Closed-ended questions:
1. What are the top 5 brands by receipts scanned among users 21 and over?

Let's assume we only use accurate data (ie transactions with products that can be found in known_products, and users that can be found in users)

```
WITH step_1 AS (
	SELECT 
        t.receipt_id, t.scan_date, t.barcode, t.user_id, p.brand, u.age
	FROM transactions t
	JOIN known_products p
	ON t.barcode = p.barcode
	JOIN users u
	ON t.user_id = u.id
)

SELECT 
    brand, COUNT(receipt_id) AS scanned_item
FROM step_1
WHERE age >= 21
GROUP BY brand
ORDER BY scanned_item DESC
LIMIT 5
```
![Alt text](data/images/query_1.png)

2. What are the top 5 brands by sales among users that have had their account for at least six months?

Let's assume we only use accurate data (ie transactions with products that can be found in known_products, and users that can be found in users)

```
WITH step_1 AS (
    SELECT 
        t.user_id, t.barcode, t.final_sale, p.brand, u.created_date
	FROM transactions t
	JOIN known_products p
	ON t.barcode = p.barcode
	JOIN users u
	ON t.user_id = u.id
)

SELECT 
    brand, SUM(final_sale) AS total_sale
FROM step_1
WHERE created_date <= NOW() - INTERVAL '29 months'
GROUP BY brand
ORDER BY total_sale DESC
LIMIT 5
```
![Alt text](data/images/query_2.png)

3. What is the percentage of sales in the Health & Wellness category by generation?

By generation, I would use the age group column that I created during EDA.

Again assume we only use accurate data (ie transactions with products that can be found in known_products, and users that can be found in users)

```
WITH step_1 AS (
    SELECT 
        t.final_sale, u.age_group, p.category_1
    FROM transactions t
    JOIN known_products p
    ON t.barcode = p.barcode
    JOIN users u
    ON t.user_id = u.id
)

SELECT 
    age_group AS generation, ROUND(CAST(SUM(final_sale)/(SELECT SUM(final_sale) FROM step_1) * 100 AS NUMERIC) , 2) AS percentage_of_sales_in_health_wellness
FROM step_1
WHERE category_1 = 'Health & Wellness'
GROUP BY age_group
ORDER BY generation
```
![Alt text](data/images/query_3.png)

Open-ended questions: for these, make assumptions and clearly state them when answering the question.

1. Who are Fetch’s power users?

To approach this problem, we have to define 'power users'. A power user is a customer that uses a company's service more often and in more effective ways. They are typically a highly engaged segment, who can be identified with usage, engagement and other behavior metrics. Given the limited data, I would like to identify the power users by examining their frequency in purchasing on Fetch, total amount they spent, and the variety of items they purchased.

To analyze and compare users more fairly, we need to normalize the data by dividing their engagement metrics by the number of months since they joined. It adjusts the metrics to account for the differences in the duration of a user's activity. I would also like to categorize the user base into new users, who created their accounts for less than a year, and loyal users, who created their accounts for more than a year. We would then be able to obtain the power users who are new to Fetch, and loyal users to Fetch.

To get the power users, we can modify the percentile to select more or less users. For now I used 80th percentile.

Again let's assume we only use accurate data (ie transactions with products that can be found in known_products, and users that can be found in users)

```
WITH all_info AS (
    SELECT 
        u.id,
        COUNT(DISTINCT t.receipt_id) AS total_unique_receipt,
        SUM(t.final_sale) total_sale,
        COUNT(DISTINCT p.category_1) total_categories,
        EXTRACT(YEAR FROM AGE(NOW(), u.created_date)) * 12 + EXTRACT(MONTH FROM AGE(NOW(), u.created_date)) AS months_since_created,
        CASE 
            WHEN EXTRACT(YEAR FROM AGE(NOW(), u.created_date)) * 12 + EXTRACT(MONTH FROM AGE(NOW(), u.created_date)) <= 12 THEN 'new'
            ELSE 'loyal'
            END AS user_category
    FROM users u
    JOIN transactions t 
    ON u.id = t.user_id
    JOIN known_products p 
    ON t.barcode = p.barcode
    GROUP BY u.id, months_since_created
),

normalized_info AS (
    SELECT 
        id, 
        user_category,
        total_unique_receipt, 
        total_sale, 
        total_categories, 
        months_since_created,
            
        -- Normalize metrics by months since account creation
        total_unique_receipt / months_since_created AS orders_per_month,
        total_sale / months_since_created AS sales_per_month,
        total_categories / months_since_created AS categories_per_month
    FROM all_info
    WHERE months_since_created > 0
),

percentiles AS (
    SELECT
        user_category,
        PERCENTILE_DISC(0.8) WITHIN GROUP (ORDER BY orders_per_month) AS top_order,
        PERCENTILE_DISC(0.8) WITHIN GROUP (ORDER BY sales_per_month) AS top_sale,
        PERCENTILE_DISC(0.8) WITHIN GROUP (ORDER BY categories_per_month) top_categories
    FROM normalized_info
    GROUP BY user_category
)

SELECT 
    n.id, n.user_category, n.orders_per_month, n.sales_per_month, n.categories_per_month
FROM normalized_info n
JOIN percentiles p
ON n.user_category = p.user_category
WHERE n.orders_per_month >= p.top_order
AND n.sales_per_month >= p.top_sale
AND n.categories_per_month >= p.top_categories
```

![Alt text](data/images/query_4.png)

2. Which is the leading brand in the Dips & Salsa category?

To determine the elading brand in the Dips & Salsa category, we can look into the total sales amount and total quantity purchased by users.

Assume we only use accurate data (ie transactions with products that can be found in known_products)

```
SELECT 
    p.brand,
    SUM(t.final_sale) AS total_sales,
	SUM(t.final_quantity) AS total_quantity
FROM transactions t
JOIN known_products p 
ON t.barcode = p.barcode
WHERE p.category_2 = 'Dips & Salsa'
GROUP BY p.brand
ORDER BY total_sales DESC
LIMIT 5;

```

![Alt text](data/images/query_5.png)

Tostitos is the leading brand in both total_sales and total_quantity in the top 5 brands in Dips & Salsa. We can insert different metrics into the growth_calculation CTE, such as user growth etc.

3. At what percent has Fetch grown year over year?

To calculate the year over year growth of Fetch, we can use different metrics. For now I'm using total sales.

```
WITH yearly_sales AS (
    SELECT 
        EXTRACT(YEAR FROM t.purchase_date) AS year,
        SUM(t.final_sale) AS total_sales
    FROM transactions t
    GROUP BY 
        EXTRACT(YEAR FROM t.purchase_date)
),

-- we can insert different metrics into the growth_calculation CTE, such as user growth etc.
growth_calculation AS (
    SELECT 
        year,
        total_sales,
        LAG(total_sales) OVER (ORDER BY year) AS previous_year_sales,
        ROUND((((total_sales - LAG(total_sales) OVER (ORDER BY year)) / NULLIF(LAG(total_sales) OVER (ORDER BY year), 0)) * 100)::NUMERIC, 2) AS yoy_growth_percentage
    FROM yearly_sales
)

SELECT * FROM growth_calculation
```
![Alt text](data/images/query_6.png)

Since we only have data in 2024 for 3 months, we cannot calculate a valid result for previous year sales and YOY growth percentage.


Interesting trend: best and worst performing day of week
```
WITH transaction_counts AS (
    SELECT 
        TO_CHAR(purchase_date, 'DAY') AS day_of_week,
        COUNT(DISTINCT receipt_id) AS order_count
    FROM 
        transactions
    GROUP BY 
        day_of_week
)


SELECT 
    'Best Performing Day' AS description,
    day_of_week,
    order_count
FROM transaction_counts
WHERE order_count = (SELECT MAX(order_count) FROM transaction_counts)
UNION ALL
SELECT 
    'Worst Performing Day' AS description,
    day_of_week,
    order_count
FROM transaction_counts
WHERE order_count = (SELECT MIN(order_count) FROM transaction_counts)
```