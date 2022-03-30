![plot](https://8weeksqlchallenge.com/images/case-study-designs/1.png)

Detailed Case Study can be accessed from [here](https://8weeksqlchallenge.com/case-study-1/)


# Schema (PostgreSQL v13)

    CREATE SCHEMA dannys_diner;
    SET search_path = dannys_diner;
    
    CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );
    
    INSERT INTO sales
      ("customer_id", "order_date", "product_id")
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
     
    
    CREATE TABLE menu (
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );
    
    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
      
    
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
    
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');

---

**Query #1**

    SELECT 
    s.customer_id as Customer,
        sum(v.price) as Total_Sales
        FROM dannys_diner.menu v
        LEFT JOIN dannys_diner.sales s 
        on v.product_id = s.product_id
        GROUP BY s.customer_id;

| customer | total_sales |
| -------- | ----------- |
| B        | 74          |
| C        | 36          |
| A        | 76          |

---
**Query #2**

    SELECT 
    customer_id,
    COUNT(sales.order_date) 
    FROM dannys_diner.sales 
    GROUP BY customer_id;

| customer_id | count |
| ----------- | ----- |
| B           | 6     |
| C           | 3     |
| A           | 6     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/485)

---
**Query #3** 


     
    

---

**Query #4**

    with ms as 
    
    (
    	select s.product_id, m.product_name from dannys_diner.sales s left join dannys_diner.menu m on s.product_id = m.product_id
    )
    
    select product_name, count(*) as c from ms group by product_name order by c desc limit 1;

| product_name | c   |
| ------------ | --- |
| ramen        | 8   |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)


---

**Query #5**

    WITH customer_cte AS (
      SELECT
        sales.customer_id,
        menu.product_name,
        COUNT(sales.*) AS item_quantity,
        RANK() OVER (
          PARTITION BY sales.customer_id
          ORDER BY COUNT(sales.*) desc
        ) AS item_rank
      FROM dannys_diner.sales
      INNER JOIN dannys_diner.menu 
      on sales.product_id = menu.product_id
      GROUP BY
        sales.customer_id,
      menu.product_name
    )
    SELECT
      customer_id,
      product_name,
      item_quantity
    FROM customer_cte
    WHERE item_rank = 1;

| customer_id | product_name | item_quantity |
| ----------- | ------------ | ------------- |
| A           | ramen        | 3             |
| B           | ramen        | 2             |
| B           | curry        | 2             |
| B           | sushi        | 2             |
| C           | ramen        | 3             |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

---

**Query #6**

    with mem as
    
    (
    	select s.customer_id, s.order_date, s.product_id, m.join_date, men.product_name, ROW_NUMBER() over(partition by s.customer_id) as rnum from dannys_diner.sales s left join dannys_diner.members m on s.customer_id = m.customer_id left join dannys_diner.menu men on s.product_id = men.product_id where s.order_date >= m.join_date order by s.order_date 
    )
    
    select customer_id, order_date, product_name from mem where rnum=1;

| customer_id | order_date               | product_name |
| ----------- | ------------------------ | ------------ |
| A           | 2021-01-07T00:00:00.000Z | curry        |
| B           | 2021-01-11T00:00:00.000Z | sushi        |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)



<!-- with mem as

(
	select s.customer_id, s.order_date, s.product_id, m.join_date, men.product_name, ROW_NUMBER() over(partition by s.order_date) as rnum from dannys_diner.sales s left join dannys_diner.members m on s.customer_id = m.customer_id left join dannys_diner.menu men on s.product_id = men.product_id where s.order_date < m.join_date order by s.order_date desc
)

select * from mem
   -->
