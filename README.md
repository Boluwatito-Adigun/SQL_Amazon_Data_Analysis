# Introduction
In today’s highly competitive e-commerce landscape, data-driven insights are essential for making informed business decisions. This project focuses on analyzing a pseudo Amazon-style transactional database designed to mirror the complexities of a real-world online marketplace. 

Through a series of simple, intermediate, and advanced SQL queries, the analysis aims to answer critical business questions that support strategic decision-making across multiple departments — including sales, inventory management, logistics, customer operations, and finance.

The goal of this analysis is to extract actionable insights that can help leadership optimize product performance, enhance customer experience, streamline operational efficiency, and ultimately maximize revenue. 

By breaking down the data into key segments such as product performance, customer behavior, shipping efficiency, and inventory health, the report provides a comprehensive understanding of the business’s current state and future opportunities.

SQL Queries? Check them out here: [project_sql_folder](/Project_scripts/)

# Background

The dataset used in this project is a simulated Amazon marketplace database, intentionally structured to reflect real e-commerce operations. Although the data is fictional, the schema, relationships, and business processes closely resemble those found in actual online retail environments. This makes the dataset highly suitable for practicing professional-level data analytics, SQL querying, and strategic business interpretation.

## Key Features of the Dataset

- **Products & Categories:** Includes product IDs, names, prices, costs, and category classifications.

- **Customers:** Contains customer demographics, registration dates, and purchasing behavior patterns.

- **Orders & Order Items:** Tracks quantities, order dates, shipping dates, payment status, and delivery information.

- **Inventory:** Holds stock levels, warehouse details, and restock history.

- **Sellers:** Features seller profiles, sales activity, and fulfillment success rates.

- **Shipping Providers:** Includes delivery partners, number of orders handled, and delivery performance metrics.

- **Returns & Cancellations:** Documents product returns, reasons, and return rate patterns.

The dataset is relational and normalized to represent how transactions flow across different business functions. This allows for meaningful end-to-end analysis — from the moment a customer registers, to placing an order, to fulfillment, delivery, and post-purchase behavior such as returns.

### Purpose of the Analysis

Using this dataset, the analysis addresses a wide range of 20+ business-critical questions designed to reveal insights such as:

- Which products and categories generate the most revenue?

- How effectively is the company retaining and monetizing customers?

- What inventory risks exist due to low stock or declining categories?

- How well are delivery providers performing?

- Which sellers and customers drive the most value?

- Where do operational inefficiencies such as shipping delays or pending shipments occur?

The queries applied range from basic aggregations to advanced analytical functions, window functions, ranking, date manipulation, and subqueries. This mirrors the type of SQL analysis expected in real e-commerce analytics roles.

# Tools Used
For this project, I employed several key tools which are;

- **SQL:** The backbone of my analysis, allowing me to query the database and discover key insights.
- **PostgresSQL:** The chosen database management system ideal for handling the job posting data
- **Git and GitHub:** Essential for version control and sharing my SQL scripts and analysis, ensuring collaboration and project tracking.  

# The Analysis
Here's how I approached each question:

## 1. Top Selling Products

This analysis identifies the 10 products generating the most revenue.
Understanding top sellers is critical because:

### Why It Matters

- These products are the core revenue drivers of the business.

- They help determine which SKUs need the most inventory investment.

- They indicate customer preferences and emerging market trends.

- They help guide promotional campaigns — promoting what already performs well leads to higher returns on marketing spend.

```sql
SELECT 
    pr.product_name,
    sum(oi.quantity) AS total_quantity,
    oi.price_per_unit,
    oi.price_per_unit * sum(oi.quantity) AS total_sales
FROM
    products pr
JOIN order_items oi 
    on pr.product_id = oi.product_id
GROUP BY
    pr.product_name,
    oi.price_per_unit
ORDER BY
    total_sales DESC
LIMIT 10;

```
## 2. Revenue by Category

This question measures how each product category contributes to total revenue.

### Why It Matters

- Categories show the broader product performance beyond individual items.

- Helps the business understand where most money is coming from.

- Highlights categories that are declining or growing.

```sql
ALTER TABLE order_items
ADD COLUMN total_sales FLOAT;

SELECT * FROM order_items

UPDATE order_items
SET total_sales = quantity * price_per_unit;


SELECT
    pr.category_id,
    c.category_name,
    SUM(oi.total_sales) AS total_revenue,
    SUM(oi.total_sales)/ 
                        (SELECT SUM(total_sales) FROM order_items) * 100 AS percentage_contribution
FROM 
    order_items oi
JOIN products pr
    ON pr.product_id = oi.product_id
LEFT JOIN category c
    ON c.category_id = pr.category_id
GROUP BY
    c.category_name,
    pr.category_id
ORDER BY
    total_revenue DESC;

```

## 3. Average Order Value (AOV) by Customer

AOV shows the average amount a customer spends per order, focusing on customers with more than five orders.

### Why It Matters

- Customers with multiple orders represent your loyal base.

- High AOV customers are highly valuable; low AOV customers may need targeted upsells.

- Helps understand spending behavior.

```sql
SELECT 
    concat (c.first_name, ' ', c.last_name) AS full_name,
    o.customer_id,
    count(o.order_id) AS number_of_orders,
    AVG(total_sales)
FROM order_items oi
JOIN orders o
    ON oi.order_id = o.order_id
JOIN customers c
     ON c.customer_id = o.customer_id
GROUP BY
    o.customer_id,
    full_name
HAVING count(o.order_id) > 5
ORDER BY
    number_of_orders DESC

```

## 4. Monthly Sales Trend

This analysis reveals how monthly revenue changes over the year, with current vs last month sales.

### Why It Matters

- Shows whether the business is growing, declining, or seasonal.

- Helps detect unusual revenue spikes or drops.

- Assists in forecasting and planning.

```sql
SELECT 
    year,
    month,
    total_sales AS current_month_sale,
    LAG(total_sales, 1) OVER(ORDER BY year, month) AS previous_month_sale
FROM
(
    SELECT 
        EXTRACT(MONTH FROM o.order_date) AS month,
        EXTRACT(YEAR FROM o.order_date) AS year,
        ROUND(SUM(oi.total_sales)::numeric, 2) AS total_sales
    FROM
        order_items oi
    JOIN orders o  
        ON oi.order_id = o.order_id
    WHERE o.order_date BETWEEN '2023-07-30' AND '2024-07-30' 
    GROUP BY
        month,
        year
    ORDER BY
        year, month
) AS sub_q1

```

## 5. Customers with No Purchases

This identifies people who registered but never placed an order.

### Why It Matters

- Shows gaps in customer conversion.

- Highlights potential issues in onboarding, pricing, website experience, or communication.

```sql
SELECT 
     concat (c.first_name, ' ', c.last_name) AS full_name,
     c.state,
     c.address
     
FROM    
    customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL
GROUP BY    
    full_name,
    c.state,
    c.address
ORDER BY 
    full_name

```