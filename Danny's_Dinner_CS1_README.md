# danny's_dinner_case_study1

## Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

### Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

##### Danny has shared with you 3 key datasets for this case study:
- sales
- menu
- members

#### DATA SET CASE STUDY1 [DOWNLOAD HERE](https://8weeksqlchallenge.com/case-study-1/)

##### Table 1: sales
The sales table captures all customer_id level purchases with an corresponding order_date and product_id information for when and what menu items were ordered.

```sql
CREATE SCHEMA dannys_diner;
CREATE DATABASE CASE_STUDY1;
USE CASE_STUDY1;
USE dannys_diner;

CREATE TABLE sales_CS1 (
customer_id VARCHAR(1),
order_date DATE,
product_id INTEGER
);

INSERT INTO sales_CS1
(customer_id, order_date, product_id)
VALUES
('A', '2021-01-01', '1'),
('A', '2021-01-01', '2'),
('A', '2021-01-07', '2'),
('A', '2021-01-10', '3'),
('A', '2021-01-11', '3'),
('A', '2021-01-11', '3'),
('B', '2021-01-01', '2'),
('B', '2021-01-02', '2'),
('B', '2021-01-04', '1'),
('B', '2021-01-11', '1'),
('B', '2021-01-16', '3'),
('B', '2021-02-01', '3'),
('C', '2021-01-01', '3'),
('C', '2021-01-01', '3'),
('C', '2021-01-07', '3');
```

##### Table 2: menu
--The menu table maps the product_id to the actual product_name and price of each menu item.
```SQL
drop table menu;
CREATE TABLE menu_CS1 (
product_id INTEGER,
product_name VARCHAR(5),
price INTEGER
);


INSERT INTO menu_CS1
(product_id, product_name, price)
VALUES
('1', 'sushi', '10'),
('2', 'curry', '15'),
('3', 'ramen', '12');
```

##### Table 3: members
--The final members table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.
```SQL
CREATE TABLE members_CS1 (
customer_id VARCHAR(1),
join_date DATE
);

INSERT INTO members_CS1
(customer_id, join_date)
VALUES
('A', '2021-01-07'),
('B', '2021-01-09');
```
	
  
  /* --------------------
   Case Study Questions
   --------------------*/
```sql
SELECT * FROM SALES_CS1;
SELECT * FROM MENU_CS1;
SELECT * FROM MEMBERS_CS1;
```
-- 1. What is the total amount each customer spent at the restaurant?
```sql
SELECT S.CUSTOMER_ID, SUM(M.PRICE) AS TOTAL_AMOUNT_FOR_EACH_CUSTOMER
FROM SALES_CS1 S
INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
GROUP BY 1;
```

### SELECT Statement:

SELECT S.CUSTOMER_ID: This selects the customer ID from the SALES_CS1 table.
SUM(M.PRICE) AS TOTAL_AMOUNT_FOR_EACH_CUSTOMER: This calculates the total amount spent by each customer by summing up the prices of the products they purchased from the MENU_CS1 table. The SUM() function is used to aggregate the prices, and it is aliased as "TOTAL_AMOUNT_FOR_EACH_CUSTOMER".
FROM Clause:

### FROM Clause:
FROM **SALES_CS1 S**: This specifies the SALES_CS1 table and aliases it as "S".
**INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID**: This performs an inner join between the SALES_CS1 table (aliased as "S") and the MENU_CS1 table (aliased as "M") based on the PRODUCT_ID column. This ensures that only the sales transactions with corresponding product information are included in the calculation.
GROUP BY Clause:

### GROUP BY 1: 
This groups the results by the first column in the SELECT statement, which is S.CUSTOMER_ID. It means that the total amount spent by each customer will be calculated separately.


### Summary :
This query retrieves the total amount spent by each customer by joining the SALES_CS1 and MENU_CS1 tables and grouping the results by customer ID.

### Output :
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/b5cc0fab-cc64-498e-9c05-63f9bc49ff13)

-- 2. How many days has each customer visited the restaurant?
```sql
SELECT CUSTOMER_ID, COUNT(DISTINCT(ORDER_DATE)) AS VISITED_COUNT_RESTAURANT
FROM SALES_CS1 
GROUP BY 1;
```
### SELECT Statement:

SELECT CUSTOMER_ID: This selects the customer ID from the SALES_CS1 table.
COUNT(DISTINCT(ORDER_DATE)) AS VISITED_COUNT_RESTAURANT: This counts the number of distinct order dates for each customer. The DISTINCT keyword ensures that each date is counted only once per customer. It's aliased as "VISITED_COUNT_RESTAURANT".

### FROM Clause:
FROM SALES_CS1: This specifies the SALES_CS1 table from which the data is being queried.

### GROUP BY Clause:
GROUP BY 1: This groups the results by the first column in the SELECT statement, which is CUSTOMER_ID. It means that the count of distinct order dates will be calculated separately for each customer.

### Summary :
This query retrieves the count of distinct order dates for each customer from the SALES_CS1 table and presents it as "VISITED_COUNT_RESTAURANT". This count represents how many days each customer visited the restaurant.

### Output :
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/11939e73-6670-4ab0-b010-8c2d14fd3f7f)

-- 3. What was the first item from the menu purchased by each customer?
```sql
WITH FIRST_ITEMS AS 
(SELECT S.CUSTOMER_ID,M.PRODUCT_NAME,S.ORDER_DATE,
DENSE_RANK() OVER (PARTITION BY CUSTOMER_ID ORDER BY ORDER_DATE ASC) AS A
FROM SALES_CS1 S
INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
GROUP BY 1,2,3)
SELECT CUSTOMER_ID,PRODUCT_NAME AS FIRST_OREDRED_ITEMS
FROM FIRST_ITEMS
WHERE A = 1;
```

### Common Table Expression (CTE) FIRST_ITEMS:

This CTE selects the customer ID, product name, and order date for each sale, and assigns a dense rank to each sale within each customer, ordered by the order date in ascending order.
DENSE_RANK() OVER (PARTITION BY CUSTOMER_ID ORDER BY ORDER_DATE ASC) AS A: This assigns a dense rank to each sale within each customer, ensuring that the first sale for each customer receives a rank of 1.
GROUP BY 1,2,3: This groups the data by the selected columns.

### Main Query:
This query selects the customer ID and product name from the FIRST_ITEMS CTE where the dense rank (A) is equal to 1, indicating the first ordered item for each customer.
WHERE A = 1: This condition filters the results to only include rows where the dense rank is 1, representing the first ordered item for each customer.

### Summary:
This query retrieves the first ordered item for each customer from the SALES_CS1 table by joining it with the MENU_CS1 table and using a dense rank to identify the first sale for each customer.

### Output :
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/d456ef2a-832f-4979-8e60-61257b29135c)


-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT M.PRODUCT_NAME, COUNT(S.PRODUCT_ID) AS NO_OF_TIMES_PURCHASED
FROM SALES_CS1 S
INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
GROUP BY 1
ORDER BY NO_OF_TIMES_PURCHASED DESC
LIMIT 1 ;
```

### SELECT Statement:

SELECT M.PRODUCT_NAME: This selects the product name from the MENU_CS1 table.
COUNT(S.PRODUCT_ID) AS NO_OF_TIMES_PURCHASED: This counts the number of times each product has been purchased from the SALES_CS1 table. It's aliased as "NO_OF_TIMES_PURCHASED".

### FROM Clause:
FROM SALES_CS1 S: This specifies the SALES_CS1 table.
INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID: This performs an inner join between the SALES_CS1 table and the MENU_CS1 table based on the PRODUCT_ID column, linking the product information to the sales transactions.

### GROUP BY Clause:
GROUP BY 1: This groups the results by the first column in the SELECT statement, which is M.PRODUCT_NAME. It means that the count of sales will be calculated for each product name.

### ORDER BY Clause:
ORDER BY NO_OF_TIMES_PURCHASED DESC: This orders the results by the count of sales in descending order, so the product that has been purchased the most will appear first.

### LIMIT 1:
This limits the result set to only one row, effectively selecting the product with the highest count of purchases.

### Summary : 
This query retrieves the product name that has been purchased the most along with the count of how many times it has been purchased from the SALES_CS1 table by joining it with the MENU_CS1 table and grouping the results by product name. It then orders the results by the count of purchases in descending order and limits the result to only one row to get the top-selling product.
### Output :
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/a739f9dd-c834-4a93-8879-01d5fc9781fe)


-- 5. Which item was the most popular for each customer?
```sql
WITH MOST_PURCHASED_ITEM AS 
	(SELECT S.CUSTOMER_ID,M.PRODUCT_NAME,COUNT(S.PRODUCT_ID) AS COUNT_OF_ORDERED,
	DENSE_RANK() OVER (PARTITION BY S.CUSTOMER_ID ORDER BY COUNT(S.PRODUCT_ID) DESC) AS COUNT_OF_ITEMS
	FROM SALES_CS1 S
	INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
	GROUP BY 1,2
	ORDER BY COUNT_OF_ORDERED DESC)
SELECT CUSTOMER_ID,PRODUCT_NAME
FROM MOST_PURCHASED_ITEM
WHERE COUNT_OF_ITEMS = 1;
```

### Common Table Expression (CTE) MOST_PURCHASED_ITEM:
This CTE calculates the count of each product purchased by each customer and assigns a dense rank to each count within each customer, ordered by the count of ordered items in descending order.
DENSE_RANK() OVER (PARTITION BY S.CUSTOMER_ID ORDER BY COUNT(S.PRODUCT_ID) DESC) AS COUNT_OF_ITEMS: This assigns a dense rank to each count of ordered items within each customer, ensuring that the most frequently purchased item for each customer receives a rank of 1.

### GROUP BY 1,2: 
This groups the data by the customer ID and product name.

### ORDER BY COUNT_OF_ORDERED DESC: 
This orders the results by the count of ordered items in descending order.

### Main Query:
This query selects the customer ID and product name from the MOST_PURCHASED_ITEM CTE where the dense rank (COUNT_OF_ITEMS) is equal to 1, representing the most frequently purchased item for each customer.
WHERE COUNT_OF_ITEMS = 1: This condition filters the results to only include rows where the dense rank is 1, indicating the most frequently purchased item for each customer.

### Summary : 
This query identifies the most frequently purchased item by each customer from the SALES_CS1 table, joined with the MENU_CS1 table to obtain the product names. It then uses a dense rank to identify the most frequently purchased item for each customer and filters the results accordingly.

### Output :
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/4e1f6392-fe9d-42b6-86dc-b17955a8c43d)


-- 6. Which item was purchased first by the customer after they became a member?
```sql
WITH AFTER_MEMBER AS 
    (SELECT S.CUSTOMER_ID,S.PRODUCT_ID,S.ORDER_DATE,M.PRODUCT_NAME,
	DENSE_RANK() OVER(PARTITION BY S.CUSTOMER_ID ORDER BY S.ORDER_DATE) AS FIRST_PRCHASED
	FROM SALES_CS1 S
	INNER JOIN MEMBERS_CS1 MEM ON S.CUSTOMER_ID = MEM.CUSTOMER_ID
	INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
	WHERE S.ORDER_DATE > MEM.JOIN_DATE) 
SELECT CUSTOMER_ID, PRODUCT_NAME AS AFTER_MEMBER_PURCHASED_FIRST_ITEM
FROM AFTER_MEMBER
WHERE FIRST_PRCHASED =1;
```

### Common Table Expression (CTE) AFTER_MEMBER:
This CTE selects relevant information for each sale made by customers after they joined as members.
DENSE_RANK() OVER(PARTITION BY S.CUSTOMER_ID ORDER BY S.ORDER_DATE) AS FIRST_PURCHASED: This assigns a dense rank to each sale made by customers after joining, ordered by the order date. This allows us to identify the first purchase made by each customer after becoming a member.
The WHERE clause ensures that only sales made after the customer's join date are included.

### Main Query:
This query selects the customer ID and product name from the AFTER_MEMBER CTE where the dense rank (FIRST_PURCHASED) is equal to 1, indicating the first purchase made by each customer after becoming a member.
WHERE FIRST_PURCHASED = 1: This condition filters the results to only include rows where the dense rank is 1, representing the first purchase made by each customer after becoming a member.

### Summary : 
This query identifies the first item purchased by each customer from the SALES_CS1 table after they became a member, by joining relevant tables and using a dense rank to identify the first purchase after joining.
### Output :
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/fe76ad5b-c8aa-4eec-bfcd-a73d6ff8b5c2)


-- 7. Which item was purchased just before the customer became a member?
```sql
WITH AFTER_MEMBER AS 
    (SELECT S.CUSTOMER_ID,S.PRODUCT_ID,S.ORDER_DATE,M.PRODUCT_NAME,
	DENSE_RANK() OVER(PARTITION BY S.CUSTOMER_ID ORDER BY S.ORDER_DATE DESC) AS FIRST_PRCHASED
	FROM SALES_CS1 S
	INNER JOIN MEMBERS_CS1 MEM ON S.CUSTOMER_ID = MEM.CUSTOMER_ID
	INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
	WHERE S.ORDER_DATE < MEM.JOIN_DATE ) 
SELECT CUSTOMER_ID, PRODUCT_NAME AS AFTER_MEMBER_PURCHASED_FIRST_ITEM
FROM AFTER_MEMBER
WHERE FIRST_PRCHASED =1;
```

### Common Table Expression (CTE) AFTER_MEMBER:

This CTE selects relevant information for each sale made by customers before they joined as members.
DENSE_RANK() OVER(PARTITION BY S.CUSTOMER_ID ORDER BY S.ORDER_DATE DESC) AS FIRST_PURCHASED: This assigns a dense rank to each sale made by customers before joining, ordered by the order date in descending order. This allows us to identify the first purchase made by each customer before becoming a member.
The WHERE clause ensures that only sales made before the customer's join date are included.

### Main Query
This query selects the customer ID and product name from the AFTER_MEMBER CTE where the dense rank (FIRST_PURCHASED) is equal to 1, indicating the first purchase made by each customer before becoming a member.

### WHERE FIRST_PURCHASED = 1: 
This condition filters the results to only include rows where the dense rank is 1, representing the first purchase made by each customer before becoming a member.

### Summary :
This modified query aims to identify the first item purchased by each customer from the SALES_CS1 table before they became a member, by joining relevant tables and using a dense rank to identify the first purchase before joining.
### Output
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/ddbf9c05-6886-4dbe-82b8-408c3a8169ff)


-- 8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT S.CUSTOMER_ID,COUNT(M.PRODUCT_NAME) AS TOTAL_ITEMS,SUM(M.PRICE)AS AMOUNT_SPENT
FROM SALES_CS1 S
INNER JOIN MEMBERS_CS1 MEM ON S.CUSTOMER_ID = MEM.CUSTOMER_ID
INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
WHERE S.ORDER_DATE < MEM.JOIN_DATE
GROUP BY 1;
```

### SELECT Statement:
SELECT S.CUSTOMER_ID: This selects the customer ID from the SALES_CS1 table.
COUNT(M.PRODUCT_NAME) AS TOTAL_ITEMS: This counts the number of distinct product names for each customer, representing the total number of items purchased by each customer.
SUM(M.PRICE) AS AMOUNT_SPENT: This calculates the total amount spent by each customer by summing up the prices of the products purchased by each customer.

### FROM Clause:
FROM SALES_CS1 S: This specifies the SALES_CS1 table.
INNER JOIN MEMBERS_CS1 MEM ON S.CUSTOMER_ID = MEM.CUSTOMER_ID: This performs an inner join between the SALES_CS1 table and the MEMBERS_CS1 table based on the CUSTOMER_ID column, linking sales transactions to membership information.
INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID: This performs an inner join between the SALES_CS1 table and the MENU_CS1 table based on the PRODUCT_ID column, linking sales transactions to product information.

### WHERE Clause:
WHERE S.ORDER_DATE < MEM.JOIN_DATE: This condition ensures that only sales made before the customer's join date are included.

### GROUP BY Clause:
GROUP BY 1: This groups the results by the first column in the SELECT statement, which is S.CUSTOMER_ID. It means that the total number of items purchased and the total amount spent will be calculated separately for each customer.

### Summary : 
This query retrieves the total number of items purchased and the total amount spent by each customer from the SALES_CS1 table before they became a member, by joining relevant tables and filtering sales made before the customer's join date.
### Output :
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/d04f3908-8ee8-4212-a8b9-a0cd8fa5c49e)


-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
SELECT S.CUSTOMER_ID , 
SUM(CASE 
	WHEN M.PRODUCT_NAME ='sushi' THEN M.PRICE * 20 
	ELSE M.PRICE * 10
    END) AS POINTS
FROM SALES_CS1 S
INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
GROUP BY 1;
```

### SELECT Statement:
SELECT S.CUSTOMER_ID: This selects the customer ID from the SALES_CS1 table.
SUM(CASE WHEN M.PRODUCT_NAME ='sushi' THEN M.PRICE * 20 ELSE M.PRICE * 10 END) AS POINTS: This calculates the total points earned by each customer. It uses a CASE statement to check if the product name is 'sushi'. If it is, it multiplies the price of the product by 20 to get the points; otherwise, it multiplies the price by 10. The SUM() function then calculates the total points earned by each customer.

### FROM Clause:
FROM SALES_CS1 S: This specifies the SALES_CS1 table.
INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID: This performs an inner join between the SALES_CS1 table and the MENU_CS1 table based on the PRODUCT_ID column, linking sales transactions to product information.

### GROUP BY Clause:
GROUP BY 1: This groups the results by the first column in the SELECT statement, which is S.CUSTOMER_ID. It means that the total points earned will be calculated separately for each customer.

### Summary : 
This query calculates the total points earned by each customer from the SALES_CS1 table, considering different points multipliers for different products, and presents the result grouped by customer ID.
### Output :
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/f2d087f6-b7a5-417c-b56a-a9a0c58bd9f3)

-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - 
how many points do customer A and B have at the end of January?
```sql
WITH OFFER_DATES AS
	(SELECT CUSTOMER_ID,JOIN_DATE, adddate(JOIN_DATE,7) AS FIRST_WEEK
	FROM MEMBERS_CS1)
SELECT S.CUSTOMER_ID,
SUM(CASE 
		WHEN (S.ORDER_DATE BETWEEN MEM.JOIN_DATE AND O.FIRST_WEEK)
        OR M.PRODUCT_NAME = 'sushi' THEN M.PRICE * 10 * 2
        ELSE M.PRICE * 10
	END) AS TOTAL_POINTS_MONTH_OF_JANUARY
FROM SALES_CS1 S
INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
INNER JOIN MEMBERS_CS1 MEM ON S.CUSTOMER_ID = MEM.CUSTOMER_ID
INNER JOIN OFFER_DATES O ON S.CUSTOMER_ID = O.CUSTOMER_ID
WHERE S.ORDER_DATE <= '2021-01-31'
GROUP BY 1
ORDER BY TOTAL_POINTS_MONTH_OF_JANUARY DESC; 
```

### Common Table Expression (CTE) OFFER_DATES:
This CTE selects the customer ID, join date, and the date exactly one week after the join date (first week after joining) from the MEMBERS_CS1 table. It uses the adddate() function to add 7 days to the join date.

### Main Query:
This query selects the customer ID and calculates the total points earned by each customer in the month of January.
SUM(CASE ...) AS TOTAL_POINTS_MONTH_OF_JANUARY: This calculates the total points earned by each customer in January. It uses a CASE statement to apply different point multipliers based on certain conditions:
If the order date is within the first week after joining or if the product name is 'sushi', the price is multiplied by 10 and then by 2 to account for a special offer. Otherwise, the price is multiplied by 10.
The SUM() function then calculates the total points earned.

### FROM Clause:
FROM SALES_CS1 S: This specifies the SALES_CS1 table.
INNER JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID: This performs an inner join between the SALES_CS1 table and the MENU_CS1 table based on the PRODUCT_ID column, linking sales transactions to product information.
INNER JOIN MEMBERS_CS1 MEM ON S.CUSTOMER_ID = MEM.CUSTOMER_ID: This performs an inner join between the SALES_CS1 table and the MEMBERS_CS1 table based on the CUSTOMER_ID column, linking sales transactions to membership information.
INNER JOIN OFFER_DATES O ON S.CUSTOMER_ID = O.CUSTOMER_ID: This performs an inner join between the SALES_CS1 table and the OFFER_DATES CTE based on the CUSTOMER_ID column, linking sales transactions to offer dates.

### WHERE Clause:
WHERE S.ORDER_DATE <= '2021-01-31': This condition filters the results to only include sales made on or before January 31, 2021.
GROUP BY Clause:

### GROUP BY 1: 
This groups the results by the first column in the SELECT statement, which is S.CUSTOMER_ID. It means that the total points earned will be calculated separately for each customer.

### ORDER BY Clause:
ORDER BY TOTAL_POINTS_MONTH_OF_JANUARY DESC: This orders the results by the total points earned in January in descending order.

### Summary : 
This query calculates the total points earned by each customer in the month of January from the SALES_CS1 table, considering different points multipliers for different products and a special offer for new members. The result is presented grouped by customer ID and ordered by the total points earned in January.
### Output
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/a848988e-d6f2-4f9b-b7bf-37f70263f491)


/*--11.JOIN ALL TABLES---*/
```sql
SELECT S.CUSTOMER_ID,S.ORDER_DATE,M.PRODUCT_NAME,M.PRICE,
CASE  
	WHEN MEM.JOIN_DATE <= S.ORDER_DATE THEN 'y'
    ELSE 'N'
END  AS MEMBERS
FROM SALES_CS1 S
LEFT JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
LEFT JOIN MEMBERS_CS1 MEM ON S.CUSTOMER_ID = MEM.CUSTOMER_ID
ORDER BY 1,2;
```

### SELECT Statement:
SELECT S.CUSTOMER_ID, S.ORDER_DATE, M.PRODUCT_NAME, M.PRICE: This selects the customer ID, order date, product name, and price from the SALES_CS1 and MENU_CS1 tables.
### CASE Statement:
CASE WHEN MEM.JOIN_DATE <= S.ORDER_DATE THEN 'Y' ELSE 'N' END AS MEMBERS: This uses a CASE statement to determine whether the customer was a member at the time of the purchase. If the join date of the customer in the MEMBERS_CS1 table is less than or equal to the order date in the SALES_CS1 table, it assigns 'Y' to indicate the customer was a member; otherwise, it assigns 'N'.

### FROM Clause:
FROM SALES_CS1 S: This specifies the SALES_CS1 table.
LEFT JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID: This performs a left join between the SALES_CS1 table and the MENU_CS1 table based on the PRODUCT_ID column, linking sales transactions to product information.
LEFT JOIN MEMBERS_CS1 MEM ON S.CUSTOMER_ID = MEM.CUSTOMER_ID: This performs a left join between the SALES_CS1 table and the MEMBERS_CS1 table based on the CUSTOMER_ID column, linking sales transactions to membership information.

### ORDER BY Clause:
ORDER BY 1, 2: This orders the results by the first and second columns in the SELECT statement, which are S.CUSTOMER_ID and S.ORDER_DATE respectively. It means that the results will be sorted first by customer ID and then by order date.

### Summary : 
This query retrieves information about sales transactions, including whether the customer was a member at the time of purchase, and presents the results ordered by customer ID and order date.
### Output
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/ebc21642-d1ec-4367-875a-35420505be41)


/*--12. Rank all the things 
Danny also requires further information about the ranking of customer products, 
but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records 
when customers are not yet part of the loyalty program.---*/
```sql
WITH RANKS AS
	(SELECT S.CUSTOMER_ID,S.ORDER_DATE,M.PRODUCT_NAME,M.PRICE,
	CASE  
		WHEN MEM.JOIN_DATE <= S.ORDER_DATE THEN 'y'
		ELSE 'N'
	END  AS MEMBERS
	FROM SALES_CS1 S
	LEFT JOIN MENU_CS1 M ON S.PRODUCT_ID = M.PRODUCT_ID
	LEFT JOIN MEMBERS_CS1 MEM ON S.CUSTOMER_ID = MEM.CUSTOMER_ID
	ORDER BY 1,2)
SELECT *, 
	CASE 
		WHEN MEMBERS = 'N' THEN NULL 
        ELSE RANK() OVER(PARTITION BY CUSTOMER_ID, MEMBERS ORDER BY ORDER_DATE)
	END AS RANK_BY_ORDER
FROM RANKS
```

### Common Table Expression (CTE) RANKS:
This CTE retrieves information about sales transactions, including the customer ID, order date, product name, price, and a flag indicating whether the customer was a member at the time of the purchase ('Y' or 'N'). It joins the SALES_CS1 table with the MENU_CS1 table to get product information and with the MEMBERS_CS1 table to access membership information. The results are ordered by customer ID and order date.

### Main Query:
The main query selects all columns from the RANKS CTE and adds an additional column for the rank by order, considering whether the customer was a member at the time of the purchase.
### CASE Statement:
CASE WHEN MEMBERS = 'N' THEN NULL ELSE RANK() OVER(PARTITION BY CUSTOMER_ID, MEMBERS ORDER BY ORDER_DATE) END AS RANK_BY_ORDER: This adds a rank to each sales transaction within each customer group, partitioned by the customer ID and the membership status ('Y' or 'N'), and ordered by the order date. If the customer was not a member at the time of the purchase, the rank is set to NULL.

### Summary :
This query retrieves information about sales transactions, including a rank for each transaction within each customer group, considering whether the customer was a member at the time of the purchase. The rank is based on the order date and is calculated separately for members and non-members.
### Output
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/7b127269-c1e4-4e52-82c7-0b349be920cc)

