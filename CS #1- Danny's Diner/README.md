# 🏮 Case Study #1: Danny's Diner

![image alt](https://8weeksqlchallenge.com/images/case-study-designs/1.png) 

## 📂 Table of Contents

- [Business Task](https://github.com/quynhddang/8-Week-SQL/edit/main/CS%20%231-%20Danny's%20Diner/README.md#business-task)
- [Entity Relationship Diagram](https://github.com/quynhddang/8-Week-SQL/edit/main/CS%20%231-%20Danny's%20Diner/README.md#entity-relationship-diagram)
- [Questions and Solutions](https://github.com/quynhddang/8-Week-SQL/edit/main/CS%20%231-%20Danny's%20Diner/README.md#entity-relationship-diagram)

All information for the case study has been sourced [here](https://8weeksqlchallenge.com/case-study-1/)

## Business Task

Danny wants to use data captured from a few months of operations to find out more about his customers, especially their visiting patterns, how much money they spent, and which menu items are their favorite. He plans to use these insights to decide whether he should expand the existing customer loyalty program- additionally he needs help to generate basic datasets so his team can easily inspect the data.
## Entity Relationship Diagram

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
WITH ordered_sales AS (
  SELECT
  	sales.customer_id,
 	  sales.order_date,
  	menu.product_name,
  	DENSE_RANK() OVER(
      PARTITION BY sales.customer_id
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
)

SELECT
	customer_id,
  product_name
FROM ordered_sales
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

- Customers A and C enjoyed ramen the most
- Customer B enjoyed all three items equally

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

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
