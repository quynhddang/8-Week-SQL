# 🏮 Case Study #1: Danny's Diner

<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/73923e95-8ef8-4703-adee-ae5751f17038" />

## 📂 Table of Contents

- [Business Task](https://github.com/quynhddang/8-Week-SQL/edit/main/CS%20%231-%20Danny's%20Diner/README.md#business-task)
- [Entity Relationship Diagram](https://github.com/quynhddang/8-Week-SQL/edit/main/CS%20%231-%20Danny's%20Diner/README.md#entity-relationshop-diagram)
- [Questions and Solutions](https://github.com/quynhddang/8-Week-SQL/edit/main/CS%20%231-%20Danny's%20Diner/README.md#entity-relationship-diagram)
- [Bonus Questions](https://github.com/quynhddang/8-Week-SQL/edit/main/CS%20%231-%20Danny's%20Diner/README.md#bonus-questions)

All information for the case study has been sourced [here](https://8weeksqlchallenge.com/case-study-1/).

## Business Task

Danny wants to use data captured from a few months of operations to find out more about his customers, especially their visiting patterns, how much money they spent, and which menu items are their favorite. He plans to use these insights to decide whether he should expand the existing customer loyalty program- additionally he needs help to generate basic datasets so his team can easily inspect the data.

## Entity Relationship Diagram

![image alt](https://github.com/quynhddang/8-Week-SQL/blob/main/CS%20%231-%20Danny's%20Diner/erd)

## Questions and Solutions

**1. What is the total amount each customer spent at the restaurant?** 

```sql
SELECT
	sales.customer_id,
	SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
```

**Answer:**

| customer_id | total_sales|
|---|---|
|A|76|
|B|74|
|C|36|

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.
  
**2. How many days has each customer visited the restaurant?**

```sql
SELECT
	customer_id,
	COUNT(DISTINCT order_date) AS number_of_visits
FROM dannys_diner.sales
GROUP BY customer_id;
```

**Answer:**

| customer_id | number_of_visits|
|---|---|
|A|4|
|B|6|
|C|2|

- Customer A has visited 4 times.
- Customer B has visited 6 times.
- Custmomer C has visited 2 times.
  
**3. What was the first item from the menu purchased by each customer?**

```sql
WITH first_orders AS (
  SELECT 
    sales.customer_id, 
    sales.order_date, 
    menu.product_name,
    DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT 
  customer_id, 
  product_name
FROM first_orders
WHERE rank = 1
GROUP BY customer_id, product_name;
```

**Answer:**

| customer_id | product_name|
|---|---|
|A|curry|
|A|sushi|
|B|curry|
|C|ramen|

- Customer A purchased both curry and sushi as their first item.
- Customer B purchased curry as their first item.
- Custmomer C purchase ramen as their first item.
  
**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT
	menu.product_name,
	COUNT(sales.product_id) AS most_purchased_item
FROM dannys_diner.menu
INNER JOIN dannys_diner.sales
	ON menu.product_id = sales.product_id
GROUP BY menu.product_name
ORDER BY most_purchased_item DESC
LIMIT 1;
```

**Answer:**

|product_name|most_purchased_item|
|---|---|
|ramen|8|

- The most purchased item is ramen, and it was purchased 8 times by all customers.
  
**5. Which item was the most popular for each customer?**

```sql
WITH most_popular AS (
	SELECT
  		sales.customer_id,
  		menu.product_name,
  		COUNT(menu.product_id) AS order_count,
  		DENSE_RANK() OVER (
			PARTITION BY sales.customer_id
			ORDER BY COUNT(sales.customer_id) DESC) AS rank
	FROM dannys_diner.sales
	INNER JOIN dannys_diner.menu
		ON sales.product_id = menu.product_id
	GROUP BY sales.customer_id, menu.product_name
)

SELECT
	customer_id,
	product_name,
	order_count
FROM most_popular
WHERE rank = 1
```

**Answer:**

|customer_id|product_name|order_count|
|---|---|---|
|A|ramen|3|
|B|ramen|2|
|B|curry|2|
|B|sushi|2|
|C|ramen|3|

- Customers A and C enjoyed ramen the most.
- Customer B enjoyed all three items equally.

**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH joined_as_member AS (
  SELECT
  	members.customer_id,
  	sales.product_id,
    ROW_NUMBER() OVER (
    PARTITION BY members.customer_id
    ORDER BY sales.order_date) AS row_number
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
	ON members.customer_id = sales.customer_id
	AND sales.order_date > members.join_date
)

SELECT
	customer_id,
	product_name
FROM joined_as_member
INNER JOIN dannys_diner.menu
  ON joined_as_member.product_id = menu.product_id
WHERE row_number = 1
ORDER BY customer_id ASC;
```

**Answers:**

|customer_id|product_name|
|---|---|
|A|ramen|
|B|sushi|

- Customer A purchased ramen first after becoming a member.
- Customer B purchased sushi first after becoming a member.

**7. Which item was purchased just before the customer became a member?**

```sql
WITH purchased_prior_member AS (
  SELECT
  	members.customer_id,
  	sales.product_id,
  	ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date DESC) AS rank
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
  	ON members.customer_id = sales.customer_id 
  	AND sales.order_date < members.join_date
)

SELECT
	prior.customer_id,
	menu.product_name
FROM purchased_prior_member AS prior
INNER JOIN dannys_diner.menu
	ON prior.product_id = menu.product_id
WHERE rank = 1 
ORDER BY prior.customer_id ASC;
```

**Answers:**

|customer_id|product_name|
|---|---|
|A|sushi|
|B|sushi|

- Before becoming a member, Customer A and B ordered sushi.
    
**8. What is the total items and amount spent for each member before they became a member?**

```sql
SELECT
	sales.customer_id,
	COUNT(sales.product_id) AS total_items,
	SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members
	ON sales.customer_id = members.customer_id
	AND sales.order_date < members.join_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

**Answers:**

|customer_id|total_items|total_sales|
|---|---|---|
|A|2|25|
|B|3|40|

- Before becoming a member, Customer A spent $25 on two items, and Customer C spent $40 on three items.
  
**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```sql
SELECT
	sales.customer_id,
	SUM(CASE
		WHEN product_name = 'sushi' THEN price*20
		ELSE price*10
		END) AS customer_points
FROM dannys_diner.menu 
INNER JOIN dannys_diner.sales  
	ON menu.product_id = sales.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

**Answers:** 

|customer_id|points|
|---|---|
|A|860|
|B|940|
|C|360|

- Customer A would have 860 points.
- Customer B would have 940 points.
- Customer C would have 360 points.
  
**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
WITH dates_cte AS (
  SELECT
  	customer_id,
  	join_date,
  	join_date + 6 AS valid_date,
  	DATE_TRUNC(
      'month', '2021-01-31'::DATE)
  		+ interval '1 month'
  		- interval '1 day' AS last_date
  FROM dannys_diner.members
)

SELECT
	sales.customer_id,
    SUM(CASE
        WHEN menu.product_name = 'sushi' THEN 2 * 10 * menu.price
        WHEN sales.order_date BETWEEN dates.join_date AND dates.valid_date THEN 2 * 10 * menu.price
        ELSE 10 * menu.price END) AS points
FROM dannys_diner.sales
INNER JOIN dates_cte as dates
	ON sales.customer_id = dates.customer_id
    AND dates.join_date <= sales.order_date
    AND sales.order_date <= dates.last_date
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
```

**Answers:**

|customer_id|points|
|---|---|
|A|1020|
|B|320|

- At the end of January, Customer A has 1020 points, and Customer B has 320 points.
  
## Bonus Questions 

**Join All The Things**

Recreate the following table output using the available data:

|customer_id|order_date|product_name|price|member_status|
|---|---|---|---|---|
|A|2021-01-01|sushi|10|N|
|A|2021-01-01|curry|15|N|
|A|2021-01-07|curry|15|Y|
|A|2021-01-10|ramen|12|Y|
|A|2021-01-11|ramen|12|Y|
|A|2021-01-11|ramen|12|Y|
|B|2021-01-01|curry|15|N|
|B|2021-01-02|curry|15|N|
|B|2021-01-04|sushi|10|N|
|B|2021-01-11|sushi|10|Y|
|B|2021-01-16|ramen|12|Y|
|B|2021-02-01|ramen|12|Y|
|C|2021-01-01|ramen|12|N|
|C|2021-01-01|ramen|12|N|
|C|2021-01-07|ramen|12|N|

**Answer:**

```sql
SELECT
	sales.customer_id,
    sales.order_date, 
    menu.product_name,
    menu.price,
    CASE
    	WHEN members.join_date > sales.order_date THEN 'N'
        WHEN members.join_date <= sales.order_date THEN 'Y'
        ELSE 'N' END AS member_status
FROM dannys_diner.sales
LEFT JOIN dannys_diner.members
	ON sales.customer_id = members.customer_id
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
ORDER BY members.customer_id, sales.order_date
```

**Rank All The Things**

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects **null** ranking values for the records when customers are not yet part of the program.

**Answer:**

```sql
WITH customers_data AS (
	SELECT
		sales.customer_id,
    	sales.order_date, 
    	menu.product_name,
    	menu.price,
    	CASE
    		WHEN members.join_Date > sales.order_date THEN 'N'
        	WHEN members.join_date <= sales.order_date THEN 'Y'
        	ELSE 'N' END AS member_status
	FROM dannys_diner.sales
	LEFT JOIN dannys_diner.members
		ON sales.customer_id = members.customer_id
	INNER JOIN dannys_diner.menu
		ON sales.product_id = menu.product_id
)

SELECT
	*,
    CASE 
    	WHEN member_status = 'N' then NULL
        ELSE RANK () OVER(
          PARTITION BY customer_id, member_status
          ORDER BY order_date
    ) END AS ranking
FROM customers_data
```

|customer_id|order_date|product_name|price|member_status|ranking|
|---|---|---|---|---|---|
|A|2021-01-01|sushi|10|N|null|
|A|2021-01-01|curry|15|N|null|
|A|2021-01-07|curry|15|Y|1|
|A|2021-01-10|ramen|12|Y|2|
|A|2021-01-11|ramen|12|Y|3|
|A|2021-01-11|ramen|12|Y|3|
|B|2021-01-01|curry|15|N|null|
|B|2021-01-02|curry|15|N|null|
|B|2021-01-04|sushi|10|N|null|
|B|2021-01-11|sushi|10|Y|1|
|B|2021-01-16|ramen|12|Y|2|
|B|2021-02-01|ramen|12|Y|3|
|C|2021-01-01|ramen|12|N|null|
|C|2021-01-01|ramen|12|N|null|
|C|2021-01-07|ramen|12|N|null|
