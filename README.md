# Customer Shopping Behavior & Segmentation - End-to-End Analytics Project

## 📌 Project Overview
This repository features an advanced, end-to-end **Customer Shopping Behavior & Segmentation Analytics** solution. This project demonstrates a full data pipeline integration: leveraging **Python (Pandas)** for data engineering, cleaning, and database extraction, **MySQL** for comprehensive relational database auditing and analytical querying, and **Power BI** for producing an interactive executive dashboard.

The primary commercial objective is to diagnose customer purchasing frequency, evaluate demographic revenue contributions, measure subscription program performance, and isolate high-value buyer cohorts.

---

## 🛠️ Technical Toolkit & Skills Demonstrated
* **Data Engineering & ETL (Python):** Utilized `pandas` for categorical missing-value imputation (handling review ratings via category medians), string sanitization, and structured feature engineering (creating `age_group` brackets via `qcut` and numerical frequency mappings).
* **Database Pipeline (SQLAlchemy & MySQL):** Configured automated database connections to write Python dataframes directly into MySQL, followed by authoring complex queries involving window functions (`ROW_NUMBER() OVER`), CTEs, subqueries, and multi-conditional logic.
* **Business Intelligence (Power BI):** Engineered a high-impact, modern visual dashboard featuring custom KPIs, cohort retention metrics, and responsive sidebar segmentation slicers.

---

## 🐍 Phase 1: Data Engineering & Cleaning (Python)
The raw shopping behavior records were processed inside a Jupyter Notebook to eliminate structural discrepancies and handle unrecorded data metrics before relational storage.
```python
# Imputing missing values in Review Rating based on product category median
df['review_rating'] = df.groupby('category')['review_rating'].transform(lambda x: x.fillna(x.median()))

# Engineering age group cohorts for localized demographic filtering
labels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']
df['age_group'] = pd.qcut(df['age'], q=4, labels=labels)

# Dropping redundant columns based on multi-column logical equality tests
df = df.drop('promo_code_used', axis=1)
```

---

## 🗄️ Phase 2: Relational Database Auditing (SQL Queries)
Once cleaned, data was loaded into a MySQL database instance (`customer_behavior`) to build enterprise reporting views. Below are core queries demonstrating deep analytical metrics:

```sql
-- 1. Identifying Premium Discounted Orders (Subqueries)
SELECT customer_id, purchase_amount
FROM customer
WHERE discount_applied = 'Yes' 
  AND purchase_amount >= (SELECT AVG(purchase_amount) FROM customer);

-- 2. Customer Cohort Segmentation (CTEs & Conditional Brackets)
WITH customer_type AS (
    SELECT customer_id, previous_purchases,
           CASE 
               WHEN previous_purchases = 1 THEN 'NEW'
               WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
               ELSE 'Loyal'
           END AS customer_segment
    FROM customer
)
SELECT customer_segment, COUNT(*) AS "Number of Customers"
FROM customer_type
GROUP BY customer_segment;

-- 3. Top 3 Most Purchased Products Per Category (Window Functions)
WITH item_counts AS (
    SELECT category, item_purchased, COUNT(customer_id) AS total_orders,
           ROW_NUMBER() OVER(PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```

---

## 📊 Phase 3: Executive Dashboard Layout (Power BI View)
<img width="1173" height="633" alt="custmoer behavior" src="https://github.com/user-attachments/assets/cfa684fc-0a1a-4a50-a585-50ee15437172" />

The visualization canvas turns verified records into high-level business insights using a distinct visual theme.

* **Core Corporate KPIs:** Houses 3 distinct cards summarizing absolute **Total Customers (3,900)**, **Average Purchase Amount (\$59.76)**, and **Average Review Rating (3.75)**.
* **Subscription & Product Categories:** Leverages high-contrast donut charts to evaluate subscription penetration (27% Yes vs. 73% No) paired with clustered bar charts ranking global revenue and sales counts by category verticals (Clothing, Accessories, Footwear, Outerwear).
* **Demographic Clusters:** Maps purchase volume density and dollar spending across horizontal charts explicitly broken down by age classifications (`Young Adult`, `Middle-aged`, `Adult`, `Senior`).




---

## 🚀 Execution Instructions
1. Navigate to your MySQL database to run the analytical queries on your schema.
2. Open the compiled project file using **Power BI Desktop**.
3. Interact with the vertical sidebar slicers to cross-filter across demographic or contract variables.
