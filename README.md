# Retail Sales Data Warehouse & Analytics Project

## Project Overview

This project is an end-to-end **SQL Server Data Warehouse** project built to consolidate sales data from multiple source systems into a clean, structured, and analytics-ready data model.

The project follows the **Medallion Architecture** approach using three layers:

- **Bronze Layer** – stores raw data from source CSV files
- **Silver Layer** – cleans, standardizes, and prepares the data
- **Gold Layer** – creates business-ready views using a star schema for reporting and analytics

The final Gold layer contains fact and dimension views that can be used for business intelligence, dashboarding, and SQL-based analysis.

---

## Business Objective

The goal of this project is to build a modern SQL Server data warehouse that consolidates sales-related data from **CRM** and **ERP** source systems.

The data warehouse enables:

- Clean and consistent reporting
- Integrated customer, product, and sales data
- Improved data quality
- Business-ready datasets for analytics
- A structured foundation for Power BI, Tableau, or SQL reporting

---

## Tools & Technologies

- **SQL Server**
- **SQL Server Management Studio**
- **T-SQL**
- **Stored Procedures**
- **Views**
- **Git & GitHub**
- **Draw.io / Diagrams.net**
- **Notion** for project planning

---

## Data Architecture

The project follows the **Medallion Architecture**:


  Source CSV Files
          |
          v
  Bronze Layer
  Raw data ingestion
          |
          v
  Silver Layer
  Data cleaning and standardization
          |
          v
  Gold Layer
  Business-ready star schema
          |
          v
  BI / Analytics / Reporting


Bronze Layer

The Bronze layer stores raw data exactly as received from the source systems.

Main purpose:

- Load raw CSV files into SQL Server
- Preserve original source structure
- Support traceability and debugging
- Avoid transformations at this stage

Silver Layer

The Silver layer contains cleaned and standardized data.

Main transformations include:
- Removing duplicates
- Handling missing values
- Fixing invalid values
- Removing unwanted spaces
- Standardizing gender, marital status, country, and product line values
- Converting incorrect data types
- Cleaning date fields
- Applying basic data quality rules

Gold Layer

The Gold layer contains the final business-ready data model.

Main purpose:
- Integrate data from multiple Silver tables
- Create fact and dimension views
- Use friendly column names
- Support analytics and reporting
- Build a star schema model

## Source Systems

This project uses data from two source systems:

1. CRM Source

    Contains customer, product, and sales transaction data.
    
    Example tables:
    - crm_cust_info
    - crm_prd_info
    - crm_sales_details

2. ERP Source

    Contains additional customer, location, and product category information.
    
    Example tables:
    - erp_cust_az12
    - erp_loc_a101
    - erp_px_cat_g1v2
  
## Data Warehouse Layers

1. Bronze Tables

  The Bronze layer contains raw tables loaded from CSV files.
  
  Examples:
  
     bronze.crm_cust_info
     bronze.crm_prd_info
     bronze.crm_sales_details
     bronze.erp_cust_az12
     bronze.erp_loc_a101
     bronze.erp_px_cat_g1v2

2. Silver Tables

  The Silver layer contains cleaned and standardized versions of the Bronze tables.
  
  Examples:
  
     silver.crm_cust_info
     silver.crm_prd_info
     silver.crm_sales_details
     silver.erp_cust_az12
     silver.erp_loc_a101
     silver.erp_px_cat_g1v2

  Each Silver table includes a metadata column:
  
     dw_create_date
  
  This column tracks when each record was inserted into the Silver layer.

3. Gold Views

  The Gold layer contains the final analytics-ready views.
  
     gold.dim_customers
     gold.dim_products
     gold.fact_sales


## ETL Process

The ETL process is divided into two main stored procedures.

Load Bronze Layer

    EXEC bronze.load_bronze;

  This procedure:
  - Truncates Bronze tables
  - Loads raw CSV files using BULK INSERT
  - Uses full-load logic
  - Tracks load duration
  - Includes error handling
  - Prints execution messages
  
  Load Silver Layer
  
    EXEC silver.load_silver;

  This procedure:
  - Truncates Silver tables
  - Reads data from Bronze tables
  - Applies data cleaning and transformation logic
  - Loads cleaned data into Silver tables
  - Tracks load duration
  - Includes error handling
  - Prints execution messages


## Key Data Transformations

1. Customer Data Cleaning
  Performed on CRM and ERP customer data:
    - Removed duplicate customer records
    - Kept the latest customer record using ROW_NUMBER()
    - Removed unwanted spaces using TRIM()
    - Standardized gender values
    - Standardized marital status values
    - Removed invalid customer ID prefixes
    - Handled missing values using Not Available
  
  Example gender standardization:
  
      CASE
          WHEN UPPER(TRIM(gen)) IN ('F', 'FEMALE') THEN 'Female'
          WHEN UPPER(TRIM(gen)) IN ('M', 'MALE') THEN 'Male'
          ELSE 'Not Available'
      END

2. Product Data Cleaning

   Performed on CRM and ERP product data:
    - Derived category ID from product key
    - Standardized product line values
    - Replaced missing product cost with zero
    - Fixed product historical end dates using LEAD()
    - Converted date/time values into proper DATE data type
    - Joined product data with category information
    
    Example product history logic:
    
        DATEADD(
            day,
            -1,
            LEAD(prd_start_dt) OVER (
                PARTITION BY prd_key
                ORDER BY prd_start_dt
            )
        )

3. Sales Data Cleaning

   Performed on CRM sales transaction data:
    - Converted integer date fields into proper date values
    - Replaced invalid dates with NULL
    - Validated date order
    - Checked product and customer referential integrity
    - Fixed invalid sales, quantity, and price values
    - Applied business rule: sales = quantity × price
    
    Example sales calculation logic:

        CASE
            WHEN sls_sales IS NULL
              OR sls_sales <= 0
              OR sls_sales != sls_quantity * ABS(sls_price)
            THEN sls_quantity * ABS(sls_price)
            ELSE sls_sales
        END AS sls_sales

## Gold Layer Star Schema

The Gold layer is modeled as a star schema.

    gold.dim_products --- gold.fact_sales --- gold.dim_customers

### Dimension Tables

1. gold.dim_customers

Contains customer descriptive information, including:

   - Customer key
   - Customer ID
   - Customer number
   - First name
   - Last name
   -  Country
   - Marital status
   - Gender
   - Birth date
   - Create date

2. gold.dim_products

Contains product descriptive information, including:
    
   - Product key
   - Product ID
   - Product number
   - Product name
   - Category
   - Subcategory
   - Maintenance flag
   - Cost
   - Product line
   - Start date
   - Fact Table

3. gold.fact_sales

Contains sales transaction information, including:
    
   - Order number
   - Product key
   - Customer key
   - Order date
   - Shipping date
   - Due date
   - Sales amount
   - Quantity
   - Price

## Surrogate Keys

Surrogate keys are generated in the Gold layer using ROW_NUMBER().

Examples:

    ROW_NUMBER() OVER (ORDER BY ci.cst_id) AS customer_key
    ROW_NUMBER() OVER (ORDER BY pn.prd_start_dt, pn.prd_key) AS product_key

These keys are used to connect fact and dimension views in the star schema.

## Data Quality Checks

This project includes reusable SQL quality checks for both Silver and Gold layers.

1. Silver Layer Checks

    The Silver quality checks validate:
    
    - Duplicate records
    - Null primary keys
    - Unwanted spaces
    - Invalid dates
    - Invalid date ranges
    - Invalid sales calculations
    - Missing or inconsistent categorical values

2. Gold Layer Checks

    The Gold quality checks validate:
  
    - Uniqueness of surrogate keys
    - Fact-to-dimension relationships
    - Referential integrity
    - Accuracy of the final star schema

Example Gold validation:

    SELECT *
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_customers c
        ON f.customer_key = c.customer_key
    WHERE c.customer_key IS NULL;

Expected result:

    No rows

## Analytics & Business Insights

After building the Gold layer star schema, SQL analysis was performed to evaluate sales performance, product performance, customer behavior, and category contribution.

### Key Business Metrics

| Metric | Value |
|---|---:|
| Total Sales | 29,356,250 |
| Total Quantity Sold | 60,423 |
| Average Price | 486 |
| Total Orders | 27,659 |
| Total Products | 295 |
| Total Customers | 18,484 |

### Category Performance

Bikes were the strongest revenue-generating category, contributing the majority of total sales.

| Category | Total Sales | Contribution |
|---|---:|---:|
| Bikes | 28,316,272 | 96.46% |
| Accessories | 700,262 | 2.39% |
| Clothing | 339,716 | 1.16% |

**Insight:**  
The business is highly dependent on the Bikes category, which generated 96.46% of total revenue. Accessories and Clothing contributed a much smaller share, which may indicate an opportunity for cross-selling or category growth.

### Top 5 Products by Revenue

| Product | Total Revenue |
|---|---:|
| Mountain-200 Black- 46 | 1,373,454 |
| Mountain-200 Black- 42 | 1,363,128 |
| Mountain-200 Silver- 38 | 1,339,394 |
| Mountain-200 Silver- 46 | 1,301,029 |
| Mountain-200 Black- 38 | 1,294,854 |

**Insight:**  
The top-performing products are all Mountain-200 bike models, showing strong demand for premium bike products.

### Lowest 5 Products by Revenue

| Product | Total Revenue |
|---|---:|
| Racing Socks- L | 2,430 |
| Racing Socks- M | 2,682 |
| Patch Kit/8 Patches | 6,382 |
| Bike Wash - Dissolver | 7,272 |
| Touring Tire Tube | 7,440 |

**Insight:**  
The lowest-revenue products are mainly accessories and maintenance items. Their low revenue may be influenced by lower selling prices, so these products should also be evaluated using quantity sold and profit margin.

### Sales Trend Over Time

Sales increased significantly during 2013, with the highest monthly sales recorded in December 2013.

| Month | Total Sales |
|---|---:|
| 2013-06 | 1,642,948 |
| 2013-10 | 1,673,261 |
| 2013-11 | 1,780,688 |
| 2013-12 | 1,874,128 |

**Insight:**  
The business showed strong growth during 2013, with sales reaching the highest point in December 2013. January 2014 appears to contain partial-month data, so it should not be treated as a full-period performance decline without further validation.

### Customer Segmentation

Customers were grouped into New, Regular, and VIP segments based on lifespan and total spending.

| Customer Segment | Total Customers |
|---|---:|
| New | 14,631 |
| Regular | 2,198 |
| VIP | 1,655 |

**Insight:**  
Most customers are classified as New, showing a large base of recent or short-lifecycle customers. VIP customers are smaller in number but represent an important high-value customer group.

### Customer Segment Performance

| Customer Segment | Total Customers | Total Sales | Average Order Value | Average Monthly Spend |
|---|---:|---:|---:|---:|
| New | 14,629 | 11,086,797 | 601 | 506 |
| VIP | 1,653 | 10,760,470 | 2,633 | 319 |
| Regular | 2,200 | 7,503,991 | 1,682 | 193 |

**Insight:**  
VIP customers generated almost the same total revenue as New customers despite being a much smaller group. They also had the highest average order value, making them an important segment for retention and loyalty strategies.

### Product Segment Performance

| Product Segment | Total Products | Total Sales | Total Quantity | Average Order Revenue |
|---|---:|---:|---:|---:|
| High-Performer | 66 | 27,643,724 | 20,322 | 1,879 |
| Mid-Range | 58 | 1,671,837 | 31,554 | 313 |
| Low-Performer | 6 | 35,697 | 8,528 | 6 |

**Insight:**  
High-performing products generated the majority of revenue, even though mid-range products sold more units. This suggests that high-performing products have much higher selling prices and are the main drivers of revenue.

### Summary of Insights

- Bikes contributed 96.46% of total sales and were the main revenue driver.
- Mountain-200 bike models dominated the top product rankings.
- Accessories and Clothing contributed a small share of revenue, creating potential cross-selling opportunities.
- Sales grew strongly during 2013 and peaked in December 2013.
- New customers formed the largest customer segment.
- VIP customers were fewer but generated high revenue and had the highest average order value.
- High-performing products generated most of the revenue, while mid-range products sold more units but produced lower revenue.
## Project Folder Structure

```text
sql-data-warehouse-project/
│
├── datasets/
│   ├── source_crm/
│   └── source_erp/
│
├── docs/
│   ├── data_architecture.png
│   ├── data_flow_diagram.png
│   ├── data_integration_diagram.png
│   ├── data_model.png
│   └── data_catalog.md
│
├── scripts/
│   ├── init_database.sql
│   │
│   ├── bronze/
│   │   ├── ddl_bronze.sql
│   │   └── proc_load_bronze.sql
│   │
│   ├── silver/
│   │   ├── ddl_silver.sql
│   │   └── proc_load_silver.sql
│   │
│   └── gold/
│       └── ddl_gold.sql
│
├── tests/
│   ├── quality_checks_silver.sql
│   └── quality_checks_gold.sql
│
└── README.md
```

## Documentation

This project includes supporting documentation:

```text
Document	                        Description
Data Architecture    	            Shows the Bronze, Silver, and Gold layer design
Data Flow Diagram	                Shows movement of data from source files to Gold views
Data Integration Diagram	        Shows relationships between CRM and ERP source tables
Data Model Diagram	                Shows the Gold layer star schema
Data Catalog	                    Describes Gold layer tables and columns
```

## Key SQL Concepts Used

This project uses several important SQL and data engineering concepts:

- Data warehouse design
- Medallion Architecture
- ETL pipelines
- Full load strategy
- BULK INSERT
- Stored procedures
- SQL views
- Data cleansing
- Data standardization
- Data normalization
- Data integration
- Surrogate keys
- Star schema modeling
- Fact and dimension tables
- Data quality checks
- Data lineage
- Data catalog documentation

## Business Value

This data warehouse provides a clean and structured foundation for business reporting.

It helps answer business questions such as:

A. What are the total sales?
B. Which products generate the most revenue?
C. Which product categories perform best?
D. Which customers contribute most to sales?
E. How do sales vary by country?
F. What is the relationship between customer demographics and sales?
G. Which product lines have stronger performance?

## What I Learned

Through this project, I learned how to:

- Design a SQL Server data warehouse from scratch
- Build Bronze, Silver, and Gold layers
- Load raw CSV files into SQL Server
- Write reusable ETL stored procedures
- Clean and standardize messy source data
- Apply data quality checks
- Build a star schema using fact and dimension views
- Generate surrogate keys
- Document data lineage and data models
- Structure a professional SQL portfolio project on GitHub

## About Me

I am Akash Deep Singh, a Data Analytics and AI graduate with hands-on experience in SQL, Python, Power BI, machine learning, and data engineering projects.

I am interested in Data Analyst, BI Analyst, Data Engineer, and Analytics Engineer roles where I can work on data pipelines, reporting, business intelligence, and analytics solutions.

LinkedIn: www.linkedin.com/in/akash-deep-singh-55747a19a
