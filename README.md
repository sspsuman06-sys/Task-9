

## GitHub Folder Structure

```
Task-9-SQL-Star-Schema/
│
├── data/
│   └── global_superstore.csv
│
├── sql/
│   ├── 01_create_tables.sql
│   ├── 02_insert_dimensions.sql
│   ├── 03_insert_fact_sales.sql
│   ├── 04_indexes.sql
│   └── 05_analysis_queries.sql
│
├── outputs/
│   └── analysis_outputs.csv
│
├── diagrams/
│   └── star_schema_diagram.png
│
└── README.md
```

---

## Star Schema Design

### Fact Table

* `fact_sales`

### Dimension Tables

* `dim_customer`
* `dim_product`
* `dim_region`
* `dim_date`

---

## 01_create_tables.sql

```sql
CREATE TABLE dim_customer (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    segment VARCHAR(50)
);

CREATE TABLE dim_product (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(150),
    category VARCHAR(50),
    sub_category VARCHAR(50)
);

CREATE TABLE dim_region (
    region_id SERIAL PRIMARY KEY,
    country VARCHAR(50),
    region VARCHAR(50),
    state VARCHAR(50),
    city VARCHAR(50)
);

CREATE TABLE dim_date (
    date_id SERIAL PRIMARY KEY,
    order_date DATE,
    year INT,
    month INT,
    day INT
);

CREATE TABLE fact_sales (
    sales_id SERIAL PRIMARY KEY,
    customer_id INT,
    product_id INT,
    region_id INT,
    date_id INT,
    sales DECIMAL(10,2),
    quantity INT,
    profit DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES dim_customer(customer_id),
    FOREIGN KEY (product_id) REFERENCES dim_product(product_id),
    FOREIGN KEY (region_id) REFERENCES dim_region(region_id),
    FOREIGN KEY (date_id) REFERENCES dim_date(date_id)
);
```

---

## 02_insert_dimensions.sql

```sql
INSERT INTO dim_customer (customer_name, segment)
SELECT DISTINCT "Customer Name", Segment FROM global_superstore;

INSERT INTO dim_product (product_name, category, sub_category)
SELECT DISTINCT "Product Name", Category, "Sub-Category" FROM global_superstore;

INSERT INTO dim_region (country, region, state, city)
SELECT DISTINCT Country, Region, State, City FROM global_superstore;

INSERT INTO dim_date (order_date, year, month, day)
SELECT DISTINCT "Order Date",
       EXTRACT(YEAR FROM "Order Date"),
       EXTRACT(MONTH FROM "Order Date"),
       EXTRACT(DAY FROM "Order Date")
FROM global_superstore;
```

---

## 03_insert_fact_sales.sql

```sql
INSERT INTO fact_sales (customer_id, product_id, region_id, date_id, sales, quantity, profit)
SELECT
    c.customer_id,
    p.product_id,
    r.region_id,
    d.date_id,
    g.Sales,
    g.Quantity,
    g.Profit
FROM global_superstore g
JOIN dim_customer c ON g."Customer Name" = c.customer_name
JOIN dim_product p ON g."Product Name" = p.product_name
JOIN dim_region r ON g.City = r.city
JOIN dim_date d ON g."Order Date" = d.order_date;
```

---

## 04_indexes.sql

```sql
CREATE INDEX idx_fact_customer ON fact_sales(customer_id);
CREATE INDEX idx_fact_product ON fact_sales(product_id);
CREATE INDEX idx_fact_region ON fact_sales(region_id);
CREATE INDEX idx_fact_date ON fact_sales(date_id);
```

---

## 05_analysis_queries.sql

```sql
-- Total Sales by Region
SELECT r.region, SUM(f.sales) AS total_sales
FROM fact_sales f
JOIN dim_region r ON f.region_id = r.region_id
GROUP BY r.region;

-- Top 5 Products by Profit
SELECT p.product_name, SUM(f.profit) AS total_profit
FROM fact_sales f
JOIN dim_product p ON f.product_id = p.product_id
GROUP BY p.product_name
ORDER BY total_profit DESC
LIMIT 5;

-- Yearly Sales Trend
SELECT d.year, SUM(f.sales) AS yearly_sales
FROM fact_sales f
JOIN dim_date d ON f.date_id = d.date_id
GROUP BY d.year
ORDER BY d.year;chaho to main **Star schema diagram**, **CSV output**, ya **MySQL/SQLite version** bhi bana deta hoon 👍
