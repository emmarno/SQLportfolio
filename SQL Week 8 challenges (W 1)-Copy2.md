# Week 1 of 8

### Challenge 1 - Danny's Diner: Through SQL analysis of sales, menu, and members datasets, we'll gain insights into customer visitation patterns, expenditure habits, and preferred menu items at Danny's Diner. These insights will inform decisions regarding enhancing customer experiences, refining loyalty programs, and potential business expansion.

1. What is the total amount each customer spent at the restaurant? 


```sql
%%sql

SELECT
customer_id, sum(price)
FROM dannys_diner.sales s 
JOIN dannys_diner.menu m2 using(product_id)
GROUP BY customer_id;
```

     * postgresql://postgres:***@localhost:5432/master
    3 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>sum</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>B</td>
            <td>74</td>
        </tr>
        <tr>
            <td>C</td>
            <td>36</td>
        </tr>
        <tr>
            <td>A</td>
            <td>76</td>
        </tr>
    </tbody>
</table>



2. How many days has each customer visited the restaurant?


```sql
%%sql 

SELECT 
customer_id, 
COUNT(DISTINCT order_date) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id;
```

     * postgresql://postgres:***@localhost:5432/master
    3 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>visit_count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>4</td>
        </tr>
        <tr>
            <td>B</td>
            <td>6</td>
        </tr>
        <tr>
            <td>C</td>
            <td>2</td>
        </tr>
    </tbody>
</table>



3. What was the first item from the menu purchased by each customer?


```sql
%%sql 

SET
  search_path = dannys_diner;
WITH ranked AS (
    SELECT
      customer_id,
      product_name,
      order_date,
      row_number() OVER (
        PARTITION BY customer_id
        ORDER BY
          order_date,
          s.product_id
      ) AS rank
    FROM
      sales AS s
      JOIN menu AS m ON s.product_id = m.product_id
  )
SELECT
  customer_id,
  product_name,
  order_date::varchar
FROM
  ranked
WHERE
  rank = 1
```

     * postgresql://postgres:***@localhost:5432/master
    Done.
    3 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>product_name</th>
            <th>order_date</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>sushi</td>
            <td>2021-01-01</td>
        </tr>
        <tr>
            <td>B</td>
            <td>curry</td>
            <td>2021-01-01</td>
        </tr>
        <tr>
            <td>C</td>
            <td>ramen</td>
            <td>2021-01-01</td>
        </tr>
    </tbody>
</table>



4. (Part 1) What is the most purchased item on the menu?


```sql
%%sql 

SET
  search_path = dannys_diner;
WITH totals AS (
    SELECT
      product_name,
      COUNT(product_name) AS total_purchase_quantity,
      row_number() OVER() AS rank
    FROM
      sales AS s
      JOIN menu AS m ON s.product_id = m.product_id
    GROUP BY
      1
  )
SELECT
  product_name,
  total_purchase_quantity
FROM
  totals
WHERE
  rank = 1
```

     * postgresql://postgres:***@localhost:5432/master
    Done.
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>product_name</th>
            <th>total_purchase_quantity</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>ramen</td>
            <td>8</td>
        </tr>
    </tbody>
</table>



4. (Part 2) How many times was it purchased by all customers?


```sql
%%sql 

SELECT
customer_id, 
product_name, 
COUNT(product_name) as total_purchase_quantity
FROM sales as s 
JOIN menu as m on s.product_id = m.product_id 
GROUP BY customer_id, product_name
ORDER BY total_purchase_quantity desc;
```

     * postgresql://postgres:***@localhost:5432/master
    7 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>product_name</th>
            <th>total_purchase_quantity</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>C</td>
            <td>ramen</td>
            <td>3</td>
        </tr>
        <tr>
            <td>A</td>
            <td>ramen</td>
            <td>3</td>
        </tr>
        <tr>
            <td>B</td>
            <td>ramen</td>
            <td>2</td>
        </tr>
        <tr>
            <td>A</td>
            <td>curry</td>
            <td>2</td>
        </tr>
        <tr>
            <td>B</td>
            <td>sushi</td>
            <td>2</td>
        </tr>
        <tr>
            <td>B</td>
            <td>curry</td>
            <td>2</td>
        </tr>
        <tr>
            <td>A</td>
            <td>sushi</td>
            <td>1</td>
        </tr>
    </tbody>
</table>



5. Which item was the most popular for each customer?


```sql
%%sql 

WITH ranked AS (
    SELECT
      customer_id,
      product_name,
      COUNT(product_name) AS total_purchase_quantity,
      rank() OVER (
        PARTITION BY customer_id
        ORDER BY
          COUNT(product_name) desc
      ) AS rank
    FROM
      sales AS s
      JOIN menu AS m ON s.product_id = m.product_id
    GROUP BY
      customer_id,
      product_name
  )
SELECT
  customer_id,
  product_name,
  total_purchase_quantity
FROM
  ranked
WHERE
  rank = 1
```

     * postgresql://postgres:***@localhost:5432/master
    5 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>product_name</th>
            <th>total_purchase_quantity</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>ramen</td>
            <td>3</td>
        </tr>
        <tr>
            <td>B</td>
            <td>sushi</td>
            <td>2</td>
        </tr>
        <tr>
            <td>B</td>
            <td>curry</td>
            <td>2</td>
        </tr>
        <tr>
            <td>B</td>
            <td>ramen</td>
            <td>2</td>
        </tr>
        <tr>
            <td>C</td>
            <td>ramen</td>
            <td>3</td>
        </tr>
    </tbody>
</table>



6. Which item was purchased first by the customer after they became a member?


```sql
%%sql 

WITH ranked AS (
    SELECT
      s.customer_id,
      order_date,
      join_date,
      product_name,
      row_number() OVER (
        PARTITION BY s.customer_id
        ORDER BY
          order_date
      ) AS rank
    FROM
      sales AS s
      JOIN members AS mm ON s.customer_id = mm.customer_id
      JOIN menu AS m ON s.product_id = m.product_id
    WHERE
      order_date >= join_date
  )
SELECT
  customer_id,
  join_date::varchar,
  order_date::varchar,
  product_name
FROM
  ranked AS r
WHERE
  rank = 1
ORDER BY
  1
```

     * postgresql://postgres:***@localhost:5432/master
    2 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>join_date</th>
            <th>order_date</th>
            <th>product_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>2021-01-07</td>
            <td>2021-01-07</td>
            <td>curry</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2021-01-09</td>
            <td>2021-01-11</td>
            <td>sushi</td>
        </tr>
    </tbody>
</table>



7. Which item was purchased just before the customer became a member?


```sql
%%sql 

WITH ranked AS (
    SELECT
      s.customer_id,
      order_date,
      join_date,
      product_name,
      rank() OVER (
        PARTITION BY s.customer_id
        ORDER BY
          order_date DESC
      ) AS rank
    FROM
      sales AS s
      JOIN members AS mm ON s.customer_id = mm.customer_id
      JOIN menu AS m ON s.product_id = m.product_id
    WHERE
      order_date < join_date
  )
SELECT
  customer_id,
  join_date::varchar,
  order_date::varchar,
  product_name
FROM
  ranked AS r
WHERE
  rank = 1
ORDER BY
  1
```

     * postgresql://postgres:***@localhost:5432/master
    3 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>join_date</th>
            <th>order_date</th>
            <th>product_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>2021-01-07</td>
            <td>2021-01-01</td>
            <td>sushi</td>
        </tr>
        <tr>
            <td>A</td>
            <td>2021-01-07</td>
            <td>2021-01-01</td>
            <td>curry</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2021-01-09</td>
            <td>2021-01-04</td>
            <td>sushi</td>
        </tr>
    </tbody>
</table>



8. What is the total items and amount spent for each member before they became a member?


```sql
%%sql 

SELECT
  s.customer_id,
  COUNT(product_name) AS total_number_of_items,
  SUM(price) AS total_purchase_amount
FROM
  sales AS s
  JOIN members AS mm ON s.customer_id = mm.customer_id
  JOIN menu AS m ON s.product_id = m.product_id
WHERE
  order_date < join_date
GROUP BY
  1
ORDER BY 1
```

     * postgresql://postgres:***@localhost:5432/master
    2 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>total_number_of_items</th>
            <th>total_purchase_amount</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>2</td>
            <td>25</td>
        </tr>
        <tr>
            <td>B</td>
            <td>3</td>
            <td>40</td>
        </tr>
    </tbody>
</table>



9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?


```sql
%%sql 
SELECT
  customer_id,
  SUM(point) AS points
FROM
  sales AS s
  JOIN (
    SELECT
      product_id,
      CASE
        WHEN product_id = 1 THEN price * 20
        ELSE price * 10
      END AS point
    FROM
      menu
  ) AS p ON s.product_id = p.product_id
GROUP BY
  1
ORDER BY
  1
```

     * postgresql://postgres:***@localhost:5432/master
    3 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>points</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>860</td>
        </tr>
        <tr>
            <td>B</td>
            <td>940</td>
        </tr>
        <tr>
            <td>C</td>
            <td>360</td>
        </tr>
    </tbody>
</table>



10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


```sql
%%sql 

WITH count_points AS (
    SELECT
      s.customer_id,
      order_date,
      join_date,
      product_name,
      SUM(point) AS point
    FROM
      sales AS s
      JOIN (
        SELECT
          product_id,
          product_name,
          CASE
            WHEN product_name = 'sushi' THEN price * 20
            ELSE price * 10
          END AS point
        FROM
          menu AS m
      ) AS p ON s.product_id = p.product_id
      JOIN members AS mm ON s.customer_id = mm.customer_id
    GROUP BY
      s.customer_id,
      order_date,
      join_date,
      product_name,
      point
  )
SELECT
  customer_id,
  SUM(
    CASE
      WHEN order_date >= join_date
      AND order_date < join_date + (7 * INTERVAL '1 day')
      AND product_name != 'sushi' THEN point * 2
      ELSE point
    END
  ) AS new_points
FROM
  count_points
WHERE
  DATE_PART('month', order_date) = 1
GROUP BY
  1
ORDER BY
  1
```

     * postgresql://postgres:***@localhost:5432/master
    2 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>new_points</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>1370</td>
        </tr>
        <tr>
            <td>B</td>
            <td>820</td>
        </tr>
    </tbody>
</table>


