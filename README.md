# Pizza Sales Data Analysis

## Introduction
This project focuses on analyzing pizza sales data to extract valuable insights, such as revenue trends, customer preferences, and order patterns. Using MySQL, the project demonstrates advanced query techniques like joins, aggregations, and window functions to support data-driven decision-making.

---

## Database Schema

### Database: `PIZZA_HUT`

### Tables:

1. **ORDERS**
   ```sql
   CREATE TABLE ORDERS (
       ORDER_ID INT NOT NULL,
       ORDER_DATE DATE NOT NULL,
       ORDER_TIME TIME NOT NULL,
       PRIMARY KEY (ORDER_ID)
   );
   ```

2. **ORDER_DETAILS**
   ```sql
   CREATE TABLE ORDER_DETAILS (
       ORDER_DETAILS_ID INT NOT NULL,
       ORDER_ID INT NOT NULL,
       PIZZA_ID TEXT NOT NULL,
       QUANTITY INT NOT NULL,
       PRIMARY KEY (ORDER_DETAILS_ID)
   );
   ```

---

## Queries

### Basic Queries

1. **Total Number of Orders Placed**
   ```sql
   SELECT COUNT(order_id) AS Total_orders FROM orders;
   ```

2. **Total Revenue from Pizza Sales**
   ```sql
   SELECT ROUND(SUM(o.quantity * p.price), 2) AS Total_revenue
   FROM order_details o
   JOIN pizzas p ON o.pizza_id = p.pizza_id;
   ```

3. **Highest-Priced Pizza**
   ```sql
   SELECT pt.name
   FROM pizza_types pt
   JOIN pizzas p ON p.pizza_type_id = pt.pizza_type_id
   WHERE p.price = (SELECT MAX(price) FROM pizzas);
   ```

4. **Most Common Pizza Size Ordered**
   ```sql
   SELECT p.size, COUNT(o.order_details_id) AS ordered
   FROM order_details o
   JOIN pizzas p ON o.pizza_id = p.pizza_id
   GROUP BY p.size
   ORDER BY ordered DESC
   LIMIT 1;
   ```

5. **Top 5 Most Ordered Pizza Types**
   ```sql
   SELECT pt.name, SUM(o.quantity) AS total_quantity
   FROM pizza_types pt
   JOIN pizzas p ON pt.pizza_type_id = p.pizza_type_id
   JOIN order_details o ON o.pizza_id = p.pizza_id
   GROUP BY pt.name
   ORDER BY total_quantity DESC
   LIMIT 5;
   ```

### Intermediate Queries

1. **Total Quantity of Each Pizza Category Ordered**
   ```sql
   SELECT pt.category, SUM(od.quantity) AS TOTAL
   FROM pizza_types pt
   JOIN pizzas p ON pt.pizza_type_id = p.pizza_type_id
   JOIN order_details od ON p.pizza_id = od.pizza_id
   GROUP BY pt.category;
   ```

2. **Order Distribution by Hour**
   ```sql
   SELECT COUNT(order_id) AS order_count, HOUR(order_time) AS hour
   FROM orders
   GROUP BY hour;
   ```

3. **Category-Wise Pizza Distribution**
   ```sql
   SELECT category, COUNT(name) AS Total
   FROM pizza_types
   GROUP BY category;
   ```

4. **Average Number of Pizzas Ordered per Day**
   ```sql
   SELECT ROUND(AVG(quantity_ordered), 2) as avg_order
   FROM (
       SELECT o.order_date, SUM(od.quantity) AS quantity_ordered
       FROM order_details od
       JOIN orders o ON od.order_id = o.order_id
       GROUP BY o.order_date
   ) AS daily_orders;
   ```

5. **Top 3 Pizza Types Based on Revenue**
   ```sql
   SELECT pt.name, SUM(od.quantity * p.price) AS revenue
   FROM pizza_types pt
   JOIN pizzas p ON p.pizza_type_id = pt.pizza_type_id
   JOIN order_details od ON od.pizza_id = p.pizza_id
   GROUP BY pt.name
   ORDER BY revenue DESC
   LIMIT 3;
   ```

### Advanced Queries

1. **Percentage Contribution of Each Pizza Type to Total Revenue**
   ```sql
   SELECT pt.category, ROUND((SUM(od.quantity * p.price) / (SELECT SUM(o.quantity * p.price) FROM order_details o JOIN pizzas p ON o.pizza_id = p.pizza_id) * 100), 2) AS revenue_percentage
   FROM pizza_types pt
   JOIN pizzas p ON p.pizza_type_id = pt.pizza_type_id
   JOIN order_details od ON od.pizza_id = p.pizza_id
   GROUP BY pt.category
   ORDER BY revenue_percentage DESC;
   ```

2. **Cumulative Revenue Over Time**
   ```sql
   SELECT ORDER_DATE, SUM(REVENUE) OVER (ORDER BY ORDER_DATE) AS CUM_REVENUE
   FROM (
       SELECT o.ORDER_DATE, SUM(od.quantity * p.price) AS REVENUE
       FROM order_details od
       JOIN pizzas p ON od.pizza_id = p.pizza_id
       JOIN orders o ON o.ORDER_ID = od.ORDER_ID
       GROUP BY o.ORDER_DATE
   ) AS sales;
   ```

3. **Top 3 Most Ordered Pizza Types by Revenue for Each Category**
   ```sql
   SELECT name, revenue
   FROM (
       SELECT category, name, revenue, RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
       FROM (
           SELECT pt.category, pt.name, SUM(od.quantity * p.price) AS revenue
           FROM pizza_types pt
           JOIN pizzas p ON pt.pizza_type_id = p.pizza_type_id
           JOIN order_details od ON od.pizza_id = p.pizza_id
           GROUP BY pt.category, pt.name
       ) AS revenue_data
   ) AS ranked_data
   WHERE rn <= 3;
   ```

---

## Conclusion
This project highlights the power of MySQL in deriving actionable insights from sales data. Advanced SQL techniques were applied to analyze trends, optimize operations, and support business strategies.
