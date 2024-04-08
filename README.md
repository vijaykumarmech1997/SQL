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

![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/b5cc0fab-cc64-498e-9c05-63f9bc49ff13)



-- 2. How many days has each customer visited the restaurant?
```sql
SELECT CUSTOMER_ID, COUNT(DISTINCT(ORDER_DATE)) AS VISITED_COUNT_RESTAURANT
FROM SALES_CS1 
GROUP BY 1;
```

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
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/7b127269-c1e4-4e52-82c7-0b349be920cc)

