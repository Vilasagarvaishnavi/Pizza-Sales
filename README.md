### Retrieve the total number of orders placed
```sql
SELECT 
    COUNT(order_id) AS total_orders
FROM
    pizza.order_details;
```

### Calculate the total revenue generated from pizza sales
```sql
SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price), 0) AS Total_revenue
FROM
    pizza.pizzas
        JOIN
    pizza.order_details ON pizzas.pizza_id = order_details.pizza_id;
```

### Identify the highest-priced pizza
```sql
SELECT 
    pizza_types.name, pizzas.price AS highest_price
FROM
    pizza.pizza_types
        JOIN
    pizza.pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
```

### Identify the most common pizza size ordered
```sql
SELECT 
    pizzas.size, COUNT(order_details.order_id) AS cnt
FROM
    pizza.pizzas
        JOIN
    pizza.order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY (pizzas.size)
ORDER BY cnt DESC
LIMIT 1;
```

### Join the necessary tables to find the total quantity of each pizza category ordered
```sql
SELECT 
    pizza_types.category, SUM(order_details.quantity) as total_sum
FROM
    pizza.pizza_t;
```

### Determine the distribution of orders by hour of the day
```sql
SELECT 
    HOUR(orders.time) AS hour,
    COUNT(order_details.order_id) AS cnt_of_orders
FROM
    pizza.orders
        JOIN
    pizza.order_details ON orders.order_id = order_details.order_id
GROUP BY HOUR(orders.time)
ORDER BY COUNT(order_details.order_id) DESC;
```

### Group the orders by date and calculate the average number of pizzas ordered per day
```sql
SELECT 
    ROUND(AVG(qnty), 0) AS Avg_quantity_per_day
FROM
    (SELECT 
        orders.date, SUM(order_details.quantity) AS qnty
    FROM
        pizza.orders
    JOIN pizza.order_details ON orders.order_id = order_details.order_id
    GROUP BY (orders.date)) AS a;
```

### Determine the top 3 most ordered pizza types based on revenue
```sql
SELECT 
    pizzas.pizza_type_id AS pizza_type,
    ROUND(SUM(pizzas.price * order_details.quantity), 0) AS price
FROM
    pizza.pizzas
        JOIN
    pizza.order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizza_type
ORDER BY SUM(pizzas.price * order_details.quantity) DESC
LIMIT 3;
```

### Calculate the percentage contribution of each pizza type to total revenue
```sql
WITH total_revenue AS 
(SELECT 
    ROUND(SUM(pizzas.price * order_details.quantity), 0) AS price
FROM
    pizza.pizzas
        JOIN
    pizza.order_details ON pizzas.pizza_id = order_details.pizza_id)
SELECT 
    pizzas.pizza_type_id,
    ROUND(SUM((pizzas.price * order_details.quantity) / (SELECT 
                    price
                FROM
                    total_revenue)) * 100, 1) AS revenue_generated
FROM
    pizza.pizzas
        JOIN
    pizza.order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.pizza_type_id
ORDER BY revenue_generated DESC;
```

### Analyze the cumulative revenue generated over time
```sql
SELECT 
    sales.date, 
    SUM(revenue) OVER(ORDER BY sales.date) AS cumm_sum 
FROM
    (SELECT 
        orders.date,
        ROUND(SUM(pizzas.price * order_details.quantity)) AS revenue
    FROM
        pizza.pizzas
            JOIN
        pizza.order_details ON pizzas.pizza_id = order_details.pizza_id
            JOIN
        pizza.orders ON orders.order_id = order_details.order_id
    GROUP BY orders.date
    ORDER BY orders.date) AS sales;
```
