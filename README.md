<h1 align="center">Dannys-Diner-SQL</h1>

## 

<h2 align="center"> Introduction</h2>

<p align="justify">Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen. Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.</p>

<h2 align="center">Problem Statement</h2>

<p align="justify">Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers. He plans on using these insights to help him decide whether he should expand the existing customer loyalty program.</p>

<h2 align="center">Case Study Questions</h2>

### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT customer_id AS [Customers], FORMAT (SUM(price), '$#,0.00') AS [Amount Spent]
   FROM menu m
   JOIN sales s ON s.product_id=m.product_id
   GROUP BY customer_id;
```
### Result Set

|Customers|Amount Spent|
|---------|------------|
|A        |$76.00|
|B        |$74.00|
|C        |$36.00|

### 2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id AS [Customers], COUNT( DISTINCT order_date) AS [Days Visited]
   FROM sales
   GROUP BY customer_id;
```

### Result Set

|Customers|Days Visited|
|---------|------------|
|A        |4|
|B        |6|
|C        |2|

### Customer Analysis:

Customers A and B are the most valuable patrons, contributing $76 and $74, respectively, compared to Customer C's $36. Customer A visits 4 times with a potential revenue of $19 per visit, while Customer B visits 6 times at $12.33 per visit, and Customer C visits 2 times at $18 per visit. Customer A demonstrates high-value transactions, Customer B is the most frequent but spends less per visit, and Customer C’s low engagement indicates room for improvement.

I recommend that Danny should enhance the loyalty program to retain high-value Customers A and B through exclusive benefits like discounts or free items. For Customer C, personalized promotions and incentives can drive more visits. Upselling strategies, such as offering combo deals or add-ons, can increase the average spend per visit, particularly for Customer B. Analyzing favorite menu items and gathering customer feedback will support tailored marketing, optimized inventory, and better overall customer satisfaction.

### 3. What was the first item from the menu purchased by each customer?

```sql
WITH customer_product AS
   (
	SELECT s.customer_id, m.product_name,
	ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) rownum
	FROM sales s
	JOIN menu m
	ON s.product_id = m.product_id
   )
   SELECT customer_id, product_name
   FROM customer_product
   WHERE rownum = 1
```

### Result Set

|Customers|Product Name|
|---------|------------|
|A        |sushi|
|B        |curry|
|C        |ramen|

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT TOP 3 product_name AS [Product Name], COUNT(product_name) AS [Number of Purchases]
   FROM sales s
   JOIN menu m
   ON m.product_id = s.product_id
   GROUP BY product_name
   ORDER BY [Number of Purchases] DESC
```

### Result Set

|Product Name|Number of Purchases|
|---------|------------|
|ramen        |8|
|curry        |4|
|sushi        |3|

### 5. Which item was the most popular for each customer?

```sql
WITH Item_count AS
   (SELECT customer_id [Customer], product_name [Product Name], COUNT(*) AS order_count,
   DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(*) DESC) as Ranking
   FROM sales s
   JOIN menu m
	ON m.product_id = s.product_id
	GROUP BY customer_id, product_name
	)
	SELECT [Customer], [Product Name]
	FROM Item_count
	WHERE Ranking = 1
```

### Result Set

|Customer|Product Name|
|---------|------------|
|A        |ramen|
|B        |sushi|
|B        |curry|
|B        |ramen|
|C        |ramen|

### Food Analyis:

Customer A's first purchase was sushi, Customer B started with curry, and Customer C chose ramen. Overall, ramen is the most purchased item (8 times), followed by curry (4 times) and sushi (3 times). For preferences, Customer A favors ramen the most, while Customer B shows an equal liking for sushi, curry, and ramen, demonstrating varied tastes. Customer C's top choice is ramen, aligning with the menu's overall popularity.

Ramen, as the most popular item, should be highlighted in promotions, potentially introducing combo deals or new variations to capitalize on its appeal. For Customer A, personalized offers on ramen can strengthen loyalty. For Customer B, who enjoys a variety of items, creating mix-and-match deals can cater to their diverse preferences. Customer C's clear preference for ramen suggests targeting them with ramen-specific promotions to increase their visits. Additionally, leveraging customer insights to experiment with new menu items inspired by popular choices can help attract new customers and maintain engagement with existing ones.

### 6. Which item was purchased first by the customer after they became a member?

```sql
WITH orders AS 
(
SELECT s.customer_id, 
m.product_name, s.order_date, 
mb.join_date,
DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date) AS rn
FROM menu m
JOIN sales s
ON m.product_id = s.product_id
JOIN members mb
ON s.customer_id = mb.customer_id
WHERE s.order_date > mb.join_date
)
SELECT customer_id, product_name
FROM orders
WHERE rn = 1
```

### Result Set

|Customers|Product Name|
|---------|------------|
|A       |ramen|
|B        |sushi|

### 7. Which item was purchased just before the customer became a member?

```sql
WITH orders AS 
(
SELECT s.customer_id, m.product_name, s.order_date, mb.join_date,
DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date DESC) AS rn
FROM menu m
JOIN sales s
ON m.product_id = s.product_id
JOIN members mb
ON s.customer_id = mb.customer_id
WHERE s.order_date < mb.join_date
)
SELECT customer_id, product_name
FROM orders
WHERE rn = 1
```

### Result Set

|Customers|Product Name|
|---------|------------|
|A       |sushi|
|A        |curry|
|B        |sushi|

### 8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT s.customer_id AS [Members], COUNT(m.product_id) AS [Total Items],
FORMAT(SUM(price), '$#,0.00') AS [Amount Spent]
FROM menu m
JOIN sales s
ON m.product_id = s.product_id
JOIN members mb
ON s.customer_id = mb.customer_id
WHERE s.order_date < mb.join_date
GROUP BY s.customer_id
```

### Result Set

|Members|Total Items|Amount Spent|
|---------|------------|---------|
|A       |2|$25.00|
|B        |3|$40.00|

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier- how many points would each customer have?

```sql
WITH customer_points AS
	(SELECT s.customer_id, m.product_id, m.price,
		CASE
			WHEN m.product_name = 'sushi' THEN m.price*10*2
			ELSE m.price*10
		END as points
	FROM sales s
	JOIN menu m
	ON s.product_id = m.product_id
	)
	SELECT customer_id [Customers], SUM(points) [Total Points]
	FROM customer_points
	GROUP BY customer_id
```

### Result Set

|Customers|Total Points|
|---------|------------|
|A       |860|
|B        |940|
|C        |360|

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi. How many points do customers A and B have at the end of January?

```sql
WITH cte AS 
(SELECT s.customer_id, m.product_name, m.price, order_date, join_date,
	CASE
		WHEN s.order_date BETWEEN mb.join_date AND DATEADD(day, 7, mb.join_date) THEN m.price*10*2
		WHEN m.product_name = 'sushi' THEN m.price*10*2
		ELSE m.price*10
		END AS points
	FROM menu m
	JOIN sales s
	ON s.product_id = m.product_id
	JOIN members mb
	ON s.customer_id = mb.customer_id
	WHERE order_date < '2021-02-01'
)
SELECT customer_id [Customers], SUM(points) AS [Total Points]
FROM cte
GROUP BY customer_id
```

### Result Set 

|Customers|Total Points|
|---------|------------|
|A       |1370|
|B        |940|

### Membership Program Analysis:

Before becoming a member, Customer A spent $25 on 2 items, with their first purchases being sushi and curry. Customer B spent $40 on 3 items, with their first purchase being sushi. After joining the loyalty program, Customer A earned 860 points based on their spending, while Customer B earned 940 points. However, after the first week as a member, Customer A accumulated 1370 points due to earning double points on all items, while Customer B remained at 940 points. Ramen was the first item purchased by both customers post-membership, with Customer A’s initial purchase being ramen and Customer B’s being sushi.

To further incentivize loyalty, I recommend Danny to promote the double points offer for new members in the first week, especially emphasizing that all menu items contribute to point accumulation. For Customer A, who benefits significantly from the 2x points multiplier, ensuring they feel rewarded for their purchases can encourage more frequent visits. Customer B, despite their high spend, could benefit from personalized offers or exclusive promotions to boost points beyond the initial membership period. Analyzing which items earn the most points for customers and offering limited-time double points on top-selling items can also drive engagement. Additionally, highlighting the rewards system and the benefits of loyalty points may attract more customers to join the program.


### Data Source

8 Weeks SQL Challenge by: [Danny Ma](https//https://8weeksqlchallenge.com/)































