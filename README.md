# 📊 Data Cleaning Pipeline using BigQuery SQL

This project demonstrates an end-to-end **data cleaning and standardization pipeline in Google BigQuery** using SQL.  
The goal is to transform raw customer order data into a clean, analytics-ready dataset suitable for BI dashboards and business reporting.

---

## 📁 Dataset Information

**Project:** `firstbq-485802`  
**Dataset:** `first_dataset`  
**Source Table:** `customer`

```sql
SELECT *
FROM `firstbq-485802.first_dataset.customer`;
```

---

## 🚨 Data Quality Problems Identified

| Column | Issue |
|--------|--------|
| `order_status` | Inconsistent spellings and formats |
| `product_name` | Multiple naming variations |
| `quantity` | Mixed text and numeric values |
| `customer_name` | Random casing |
| Records | Duplicate customer-product combinations |

---

## ✅ Data Cleaning Steps

---

### ✅ Step 1: Standardize Order Status

Maps inconsistent text values into standard business categories.

```sql
SELECT order_status,
  CASE
    WHEN LOWER(order_status) LIKE '%deliver%' THEN 'Delivered'
    WHEN LOWER(order_status) LIKE '%pend%'    THEN 'Pending'
    WHEN LOWER(order_status) LIKE '%ship%'    THEN 'Shipped'
    WHEN LOWER(order_status) LIKE '%return%'  THEN 'Returned'
    WHEN LOWER(order_status) LIKE '%refund%'  THEN 'Refunded'
    ELSE 'Other'
  END AS cleaned_order_status
FROM `firstbq-485802.first_dataset.customer`;
```

---

### ✅ Step 2: Standardize Product Names

Groups multiple product variants under a single standardized product label.

```sql
SELECT product_name,
  CASE
    WHEN LOWER(product_name) LIKE '%apple watch%'    THEN 'Apple Watch'
    WHEN LOWER(product_name) LIKE '%google pixel%'   THEN 'Google Pixel'
    WHEN LOWER(product_name) LIKE '%samsung galaxy%' THEN 'Samsung Galaxy S22'
    WHEN LOWER(product_name) LIKE '%iphone%'         THEN 'iPhone 14'
    WHEN LOWER(product_name) LIKE '%macbook%'        THEN 'Macbook Pro'
    ELSE 'Other'
  END AS cleaned_product_name
FROM `firstbq-485802.first_dataset.customer`;
```

---

### ✅ Step 3: Clean and Convert Quantity

Converts text values like "two" into numeric format and casts remaining values to integers.

```sql
SELECT quantity,
  CASE
    WHEN LOWER(quantity) LIKE '%two%' THEN 2
    ELSE CAST(quantity AS INT64)
  END AS cleaned_quanity
FROM `firstbq-485802.first_dataset.customer`;
```

---

### ✅ Step 4: Standardize Customer Names

Converts names into Proper Case format.

```sql
SELECT customer_name,
  INITCAP(customer_name) AS new_customer_name
FROM `firstbq-485802.first_dataset.customer`
WHERE customer_name IS NOT NULL;
```

---

### ✅ Step 5: Remove Duplicate Records

Keeps only one record per customer email and product combination.

```sql
SELECT *
FROM (
  SELECT
    ROW_NUMBER() OVER(
      PARTITION BY LOWER(email), LOWER(product_name)
      ORDER BY order_id
    ) AS rn,
    *
  FROM `firstbq-485802.first_dataset.customer`
)
WHERE rn = 1;
```

---

## 🧩 Final Cleaned Dataset — End-to-End Query

This query performs all cleaning operations and removes duplicates in a single pipeline.

```sql
WITH cleaned_data AS (
  SELECT
    order_id,
    INITCAP(customer_name) AS new_customer_name,
    email,

    CASE
      WHEN LOWER(product_name) LIKE '%apple watch%'    THEN 'Apple Watch'
      WHEN LOWER(product_name) LIKE '%google pixel%'   THEN 'Google Pixel'
      WHEN LOWER(product_name) LIKE '%samsung galaxy%' THEN 'Samsung Galaxy S22'
      WHEN LOWER(product_name) LIKE '%iphone%'         THEN 'iPhone 14'
      WHEN LOWER(product_name) LIKE '%macbook%'        THEN 'Macbook Pro'
      ELSE 'Other'
    END AS cleaned_product_name,

    CASE
      WHEN LOWER(quantity) LIKE '%two%' THEN 2
      ELSE CAST(quantity AS INT64)
    END AS cleaned_quanity,

    price,
    INITCAP(country) AS country,

    CASE
      WHEN LOWER(order_status) LIKE '%deliver%' THEN 'Delivered'
      WHEN LOWER(order_status) LIKE '%pend%'    THEN 'Pending'
      WHEN LOWER(order_status) LIKE '%ship%'    THEN 'Shipped'
      WHEN LOWER(order_status) LIKE '%return%'  THEN 'Returned'
      WHEN LOWER(order_status) LIKE '%refund%'  THEN 'Refunded'
      ELSE 'Other'
    END AS order_status

  FROM `firstbq-485802.first_dataset.customer`
  WHERE customer_name IS NOT NULL
),

deduplicated_data AS (
  SELECT
    *,
    ROW_NUMBER() OVER(
      PARTITION BY email, cleaned_product_name
      ORDER BY order_id
    ) AS rn
  FROM cleaned_data
)

SELECT *
FROM deduplicated_data
WHERE rn = 1;
```

---

## 📤 Final Output Columns

| Column | Description |
|--------|------------|
| `order_id` | Unique order identifier |
| `new_customer_name` | Cleaned customer name |
| `email` | Customer email |
| `cleaned_product_name` | Standardized product |
| `cleaned_quanity` | Numeric quantity |
| `price` | Product price |
| `country` | Normalized country |
| `order_status` | Standardized order status |

---

## 🚀 Use Cases

- Business intelligence dashboards  
- Revenue and product performance analysis  
- Customer behavior analytics  
- Marketing and operations reporting  

---

## 🔧 Future Improvements

- Use `SAFE_CAST()` to avoid runtime casting failures  
- Persist results as a BigQuery View or Table  
- Add automated data quality validation checks  
- Integrate with scheduled ELT pipelines (dbt / Airflow)  

---

## 👩‍💻 Author

**Priya — Data Analyst**  
Skills: BigQuery SQL, Data Cleaning, ETL, BI Reporting, Analytics Engineering

---
