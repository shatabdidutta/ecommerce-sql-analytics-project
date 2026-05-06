# ecommerce-sql-analytics-project
# 🛒 E-commerce Analytics using BigQuery (SQL)

---

## 📌 Overview

This repository presents an end-to-end analytics workflow for an e-commerce platform, implemented in **Google BigQuery** using advanced SQL. The objective is to transform raw transactional data into actionable insights that inform customer strategy, revenue optimization, and product decisions.

The project is designed to mirror real-world data scenarios, combining **data modeling (star schema)** with **analytical SQL** (joins, aggregations, and window functions) to solve business-critical questions.

---

## 🎯 Business Objectives

* Quantify customer engagement and purchasing behavior
* Analyze revenue trends over time and across segments
* Evaluate product and category performance
* Identify high-value customers and revenue drivers
* Detect drop-offs in the customer lifecycle

---

## 🧱 Data Architecture

### 🔹 Schema Design: Star Schema

The data model follows a **star schema**, enabling efficient analytical queries.

* **Fact Table**

  * `orders`: transactional order-level data

* **Dimension Tables**

  * `customers`: customer attributes
  * `products`: product metadata

* **Bridge Table**

  * `order_items`: item-level granularity linking orders and products

```id="9kcm2p"
customers ──┐
            ├── orders ─── order_items ─── products
```

This structure supports scalable analytics and aligns with modern data warehouse design principles.

---

## 📂 Dataset Specification

### customers

| Column      | Type   |
| ----------- | ------ |
| customer_id | INT    |
| name        | STRING |
| city        | STRING |
| signup_date | DATE   |

### orders

| Column       | Type  |
| ------------ | ----- |
| order_id     | INT   |
| customer_id  | INT   |
| order_date   | DATE  |
| order_amount | FLOAT |

### order_items

| Column        | Type  |
| ------------- | ----- |
| order_item_id | INT   |
| order_id      | INT   |
| product_id    | INT   |
| quantity      | INT   |
| price         | FLOAT |

### products

| Column       | Type   |
| ------------ | ------ |
| product_id   | INT    |
| product_name | STRING |
| category     | STRING |
| price        | FLOAT  |

---

## ⚙️ Analytical Approach

The analysis leverages:

* **Joins** to combine transactional and dimensional data
* **Aggregations** for KPI computation
* **Window functions** to analyze trends, rankings, and cumulative behavior

---

## 📊 Key Analyses & Insights

### 1. Customer Order Frequency

**Objective:** Measure engagement level per customer

```sql
SELECT customer_id, COUNT(order_id) AS total_orders
FROM ecommerce_project.orders_large
GROUP BY customer_id;
```

**Insight:** Customer activity is unevenly distributed, with a mix of one-time and repeat buyers
**Action:** Implement retention strategies targeting one-time purchasers

---

### 2. Daily Revenue Trends

**Objective:** Identify temporal revenue patterns

```sql
SELECT order_date, SUM(order_amount) AS daily_revenue
FROM ecommerce_project.orders_large
GROUP BY order_date
ORDER BY order_date;
```

**Insight:** Revenue fluctuates across days, indicating demand variability
**Action:** Replicate high-performing day strategies and optimize low-performing periods

---

### 3. Average Order Value (AOV)

**Objective:** Evaluate typical transaction size

```sql
SELECT AVG(order_amount) AS avg_order_value
FROM ecommerce_project.orders_large;
```

**Insight:** Provides a baseline for customer spend
**Action:** Increase AOV via bundling, upselling, and pricing strategies

---

### 4. Geographic Demand Distribution

**Objective:** Analyze order distribution by city

```sql
SELECT c.city, COUNT(o.order_id) AS total_orders
FROM ecommerce_project.customers_large c
JOIN ecommerce_project.orders_large o
ON c.customer_id = o.customer_id
GROUP BY c.city;
```

**Insight:** Demand is relatively balanced, with slight concentration in key cities
**Action:** Scale high-performing markets and optimize underperforming ones

---

### 5. Category-Level Revenue

**Objective:** Identify top-performing product categories

```sql
SELECT p.category, SUM(oi.quantity * oi.price) AS revenue
FROM ecommerce_project.order_items_large oi
JOIN ecommerce_project.products_large p
ON oi.product_id = p.product_id
GROUP BY p.category;
```

**Insight:** Revenue concentration in specific categories indicates demand skew
**Action:** Prioritize inventory, pricing, and marketing for top categories

---

### 6. Product Demand Analysis

**Objective:** Measure units sold per product

```sql
SELECT product_id, SUM(quantity) AS total_sold
FROM ecommerce_project.order_items_large
GROUP BY product_id;
```

**Insight:** Demand is moderately distributed across products
**Action:** Promote high-performing products; optimize underperformers

---

### 7. Customer Conversion Gap

**Objective:** Identify customers who never transacted

```sql
SELECT c.customer_id
FROM ecommerce_project.customers_large c
LEFT JOIN ecommerce_project.orders_large o
ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

**Insight:** Significant drop-off between acquisition and conversion
**Action:** Improve onboarding, incentives, and engagement funnels

---

### 8. High-Value Transactions

**Objective:** Identify maximum spend per customer

```sql
SELECT customer_id, MAX(order_amount) AS highest_order
FROM ecommerce_project.orders_large
GROUP BY customer_id;
```

**Insight:** Presence of high-value customer segment
**Action:** Target with loyalty programs and premium offerings

---

### 9. Basket Size Analysis

**Objective:** Evaluate items per order

```sql
SELECT order_id, SUM(quantity) AS total_items
FROM ecommerce_project.order_items_large
GROUP BY order_id;
```

**Insight:** Mix of small and bulk purchases
**Action:** Increase basket size via bundles and threshold-based incentives

---

### 10. Order Value Ranking

**Objective:** Rank orders by value

```sql
SELECT order_id, order_amount,
RANK() OVER (ORDER BY order_amount DESC) AS rank
FROM ecommerce_project.orders_large;
```

**Insight:** High-value orders cluster near upper thresholds
**Action:** Introduce premium pricing tiers and upsell strategies

---

### 11. Customer Lifetime Value Trend

**Objective:** Track cumulative spend

```sql
SELECT customer_id, order_date, order_amount,
SUM(order_amount) OVER (
    PARTITION BY customer_id ORDER BY order_date
) AS running_total
FROM ecommerce_project.orders_large;
```

**Insight:** Highlights growth trajectory of customer value
**Action:** Focus retention on high-growth customers

---

### 12. Customer Journey Sequencing

**Objective:** Track order sequence per customer

```sql
SELECT customer_id, order_id, order_date,
ROW_NUMBER() OVER (
    PARTITION BY customer_id ORDER BY order_date
) AS order_number
FROM ecommerce_project.orders_large;
```

**Insight:** Distinguishes first-time vs repeat behavior
**Action:** Segment lifecycle-based marketing strategies

---

### 13. Revenue Contribution Analysis

**Objective:** Measure contribution of each order

```sql
SELECT customer_id, order_id, order_amount,
order_amount / SUM(order_amount) OVER (PARTITION BY customer_id) AS contribution
FROM ecommerce_project.orders_large;
```

**Insight:** Some customers rely heavily on single transactions
**Action:** Encourage consistent purchasing behavior

---

## 📸 Results & Visualization

Query outputs are documented in the `/screenshots` directory.

```id="d5o38k"
screenshots/
├── query_01.png
├── query_02.png
...
```

---

## 💼 Business Impact

This project demonstrates how SQL-driven analytics can:

* Improve customer retention strategies
* Optimize revenue generation
* Identify high-value segments
* Enhance product and pricing decisions

---

## 🛠 Tech Stack

* **Data Warehouse:** Google BigQuery
* **Language:** SQL
* **Modeling:** Star Schema

---

## 📌 Conclusion

This project showcases the application of advanced SQL techniques to solve real-world business problems. It reflects a strong foundation in data modeling, analytical thinking, and the ability to translate data into actionable insights.

---

## 🔗 Future Enhancements

* Dashboard integration (Tableau / Power BI)
* Customer segmentation (RFM analysis)
* Predictive modeling for demand forecasting

---
Author
Shatabdi Dutta
