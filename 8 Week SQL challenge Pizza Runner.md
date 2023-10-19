```python
from sqlalchemy import create_engine
%reload_ext sql
```


```python
%sql postgresql://postgres:***@localhost:5432/master
```

How many pizzas were ordered?


```sql
%%sql 

SELECT COUNT(*) 
FROM  pizza_runner.customer_orders co
```

     * postgresql://postgres:***@localhost:5432/master
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>14</td>
        </tr>
    </tbody>
</table>



How many unique customer orders were made? 



```sql
%%sql 

select distinct order_id
from pizza_runner.customer_orders
order BY order_id;
```

     * postgresql://postgres:***@localhost:5432/master
    10 rows affected.
    




<table>
    <thead>
        <tr>
            <th>order_id</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
        </tr>
        <tr>
            <td>2</td>
        </tr>
        <tr>
            <td>3</td>
        </tr>
        <tr>
            <td>4</td>
        </tr>
        <tr>
            <td>5</td>
        </tr>
        <tr>
            <td>6</td>
        </tr>
        <tr>
            <td>7</td>
        </tr>
        <tr>
            <td>8</td>
        </tr>
        <tr>
            <td>9</td>
        </tr>
        <tr>
            <td>10</td>
        </tr>
    </tbody>
</table>



How many successful orders were delivered by each runner?


```sql
%%sql 


select count(order_id) as successful_orders, runner_id 
from pizza_runner.runner_orders ro 
where distance notnull 
group by runner_id
order by runner_id;
```

     * postgresql://postgres:***@localhost:5432/master
    3 rows affected.
    




<table>
    <thead>
        <tr>
            <th>successful_orders</th>
            <th>runner_id</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>4</td>
            <td>1</td>
        </tr>
        <tr>
            <td>4</td>
            <td>2</td>
        </tr>
        <tr>
            <td>2</td>
            <td>3</td>
        </tr>
    </tbody>
</table>



How many of each type of pizza was delivered?


```sql
%%sql 

SELECT 
pizza_runner.pizza_names.pizza_name, 
       COUNT(c.pizza_id) AS delivered_pizza_count
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r 
  ON c.order_id = r.order_id
JOIN pizza_runner.pizza_names 
  ON c.pizza_id = pizza_runner.pizza_names.pizza_id
WHERE r.distance IS NOT NULL
GROUP BY pizza_runner.pizza_names.pizza_name;
```

     * postgresql://postgres:***@localhost:5432/master
    2 rows affected.
    




<table>
    <thead>
        <tr>
            <th>pizza_name</th>
            <th>delivered_pizza_count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Meatlovers</td>
            <td>10</td>
        </tr>
        <tr>
            <td>Vegetarian</td>
            <td>4</td>
        </tr>
    </tbody>
</table>



How many Vegetarian and Meatlovers were ordered by each customer?


```sql
%%sql 

SELECT pizza_runner.pizza_names.pizza_name, 
       c.customer_id,
       COUNT(pizza_runner.pizza_names.pizza_id) AS delivered_pizza_count
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r 
  ON c.order_id = r.order_id
JOIN pizza_runner.pizza_names 
  ON c.pizza_id = pizza_runner.pizza_names.pizza_id
WHERE r.distance IS NOT NULL
GROUP BY pizza_runner.pizza_names.pizza_name, c.customer_id;
```

     * postgresql://postgres:***@localhost:5432/master
    8 rows affected.
    




<table>
    <thead>
        <tr>
            <th>pizza_name</th>
            <th>customer_id</th>
            <th>delivered_pizza_count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Meatlovers</td>
            <td>103</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Vegetarian</td>
            <td>105</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Meatlovers</td>
            <td>101</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Vegetarian</td>
            <td>103</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Meatlovers</td>
            <td>102</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Vegetarian</td>
            <td>101</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Meatlovers</td>
            <td>104</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Vegetarian</td>
            <td>102</td>
            <td>1</td>
        </tr>
    </tbody>
</table>



What was the maximum number of pizzas delivered in a single order?


```sql
%%sql 

SELECT 
  MAX(pizza_per_order) AS pizza_count
FROM (
  SELECT 
    c.order_id, 
    COUNT(c.pizza_id) AS pizza_per_order
  FROM pizza_runner.customer_orders AS c
  JOIN pizza_runner.runner_orders AS r
    ON c.order_id = r.order_id
  WHERE r.distance notnull 
  GROUP BY c.order_id
) AS pizza_counts;
```

     * postgresql://postgres:***@localhost:5432/master
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>pizza_count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>3</td>
        </tr>
    </tbody>
</table>



For each customer, how many delivered pizzas had at least 1 change and how many had no changes?


```sql
%%sql 


SELECT 
  c.customer_id
  ,SUM(
    CASE WHEN c.exclusions != '' OR c.extras != '' THEN 1
    ELSE 0
    END) AS at_least_1_change,
  SUM(
    CASE WHEN c.exclusions = '' AND c.extras = '' THEN 1 
    ELSE 0
    END) AS no_change
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r
  ON c.order_id = r.order_id
WHERE r.duration != ''
GROUP BY c.customer_id
ORDER BY c.customer_id;
```

     * postgresql://postgres:***@localhost:5432/master
    5 rows affected.
    




<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>at_least_1_change</th>
            <th>no_change</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>101</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>102</td>
            <td>1</td>
            <td>1</td>
        </tr>
        <tr>
            <td>103</td>
            <td>4</td>
            <td>0</td>
        </tr>
        <tr>
            <td>104</td>
            <td>3</td>
            <td>0</td>
        </tr>
        <tr>
            <td>105</td>
            <td>1</td>
            <td>0</td>
        </tr>
    </tbody>
</table>



How many pizzas were delivered that had both exclusions and extras?


```sql
%%sql
SELECT  
  SUM(
    CASE WHEN exclusions <> '' AND extras <> '' THEN 1
    ELSE 0
    END) AS pizza_count_w_exclusions_extras
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r
  ON c.order_id = r.order_id
WHERE 
  r.distance ~ E'^\\d+\\.?\\d*$' -- Check if distance is numeric
  AND CAST(SPLIT_PART(r.distance, 'km', 1) AS NUMERIC) >= 1 
  AND exclusions <> '' 
  AND extras <> '';
```

     * postgresql://postgres:***@localhost:5432/master
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>pizza_count_w_exclusions_extras</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
        </tr>
    </tbody>
</table>



What was the total volume of pizzas ordered for each hour of the day?


```sql
%%sql 

SELECT 
DATE_PART('HOUR', order_time) 
AS hour_of_day, 
COUNT(order_id) AS pizza_count
FROM pizza_runner.customer_orders
GROUP BY DATE_PART('HOUR', order_time)
ORDER BY hour_of_day;
```

     * postgresql://postgres:***@localhost:5432/master
    6 rows affected.
    




<table>
    <thead>
        <tr>
            <th>hour_of_day</th>
            <th>pizza_count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>11.0</td>
            <td>1</td>
        </tr>
        <tr>
            <td>13.0</td>
            <td>3</td>
        </tr>
        <tr>
            <td>18.0</td>
            <td>3</td>
        </tr>
        <tr>
            <td>19.0</td>
            <td>1</td>
        </tr>
        <tr>
            <td>21.0</td>
            <td>3</td>
        </tr>
        <tr>
            <td>23.0</td>
            <td>3</td>
        </tr>
    </tbody>
</table>



What was the volume of orders for each day of the week?


```sql
%%sql 

SELECT 
    TO_CHAR(order_time + INTERVAL '2 days', 'Day') AS day_of_week,
    COUNT(order_id) AS total_pizzas_ordered
FROM pizza_runner.customer_orders
GROUP BY TO_CHAR(order_time + INTERVAL '2 days', 'Day')
```

     * postgresql://postgres:***@localhost:5432/master
    4 rows affected.
    




<table>
    <thead>
        <tr>
            <th>day_of_week</th>
            <th>total_pizzas_ordered</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Saturday </td>
            <td>3</td>
        </tr>
        <tr>
            <td>Sunday   </td>
            <td>1</td>
        </tr>
        <tr>
            <td>Monday   </td>
            <td>5</td>
        </tr>
        <tr>
            <td>Friday   </td>
            <td>5</td>
        </tr>
    </tbody>
</table>



How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)


```sql
%%sql 

SELECT 
  EXTRACT(WEEK FROM registration_date) AS registration_week,
  COUNT(runner_id) AS runner_signup
FROM pizza_runner.runners
GROUP BY EXTRACT(WEEK FROM registration_date);
```

     * postgresql://postgres:***@localhost:5432/master
    3 rows affected.
    




<table>
    <thead>
        <tr>
            <th>registration_week</th>
            <th>runner_signup</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>53.0</td>
            <td>4</td>
        </tr>
        <tr>
            <td>1.0</td>
            <td>2</td>
        </tr>
        <tr>
            <td>2.0</td>
            <td>2</td>
        </tr>
    </tbody>
</table>


