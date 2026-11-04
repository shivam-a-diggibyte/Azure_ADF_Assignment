# Azure Data Factory and Databricks – Sales View Project

## Project Overview

This project demonstrates an end-to-end data engineering pipeline developed using Azure Data Factory, Azure Data Lake Storage Gen2, and Azure Databricks.

The objective is to ingest Customer, Product, Store, and Sales datasets, identify the latest modified source files, perform data profiling and data transformations, and generate a consolidated Gold dataset for business analysis.

The solution follows the Medallion Architecture (Bronze, Silver, and Gold).

## Technology Stack

- Azure Data Factory (ADF)
- Azure Data Lake Storage Gen2 (ADLS Gen2)
- Azure Databricks
- PySpark
- Delta Lake
- Unity Catalog
- Azure Managed Identity
- Python

## Architecture Overview

```text
Source CSV Files (ADLS Gen2)
           |
           v
Azure Data Factory
           |
           v
Bronze Layer (Raw Data)
           |
           v
Azure Databricks (PySpark)
           |
           v
Silver Layer (Cleaned Data)
           |
           v
Product + Store + Sales Integration
           |
           v
Gold Layer (Business-Ready Data)
           |
           v
Sales View Analysis
```

## Source Datasets

The project uses four source datasets:

| Dataset | Description |
|---|---|
| Customer | Customer details, email, gender, joining date, and expenditure |
| Product | Product details, category, pricing, stock, and store references |
| Store | Store information, location, manager, and contact details |
| Sales | Order details, customer and product references, sales, and profit |

## Azure Data Factory – Data Ingestion

A pipeline named `PL_Latest_File_To_Bronze` was developed to dynamically identify and copy the latest modified file from each source folder.

### Activities Implemented

- Get Metadata
- Filter
- ForEach
- Get Metadata – Last Modified
- If Condition
- Set Variable
- Copy Data

The pipeline compares file modification timestamps and copies the latest modified file into the corresponding Bronze folder.

## Medallion Architecture Implementation

### 1. Bronze Layer

The Bronze layer stores raw CSV files ingested through Azure Data Factory.

```text
bronze/
├── customer/
├── product/
├── store/
└── sales/
```

### 2. Silver Layer

Azure Databricks and PySpark were used to clean and transform the datasets.

**Customer Transformations**
- Standardized column names into lowercase snake_case.
- Split customer names into `first_name` and `last_name`.
- Extracted email domains.
- Standardized gender values into `M` and `F`.
- Separated joining date and time.
- Standardized date formats.
- Created `expenditure_status` based on spending.

**Product Transformations**
- Standardized column names.
- Created `sub_category` based on `category_id`.
- Standardized product date columns.

**Store Transformations**
- Standardized column names.
- Extracted `store_category` from email addresses.
- Standardized `created_at` and `updated_at`.

**Sales Transformations**
- Standardized column names.
- Standardized `order_date` and `ship_date`.

### Silver Delta Tables

```text
silver/
└── sales_view/
    ├── customer/
    ├── product/
    ├── store/
    └── customer_sales/
```

Delta Lake MERGE operations were implemented to support inserting new records and updating existing records.

| Dataset | Merge Key |
|---|---|
| Customer | `customer_id` |
| Product | `product_id` |
| Store | `store_id` |
| Sales | `order_id`, `product_id` |

### 3. Gold Layer

The Gold layer integrates Product, Store, and Sales information into a consolidated dataset.

**Integration Process**

1. Product and Store datasets are joined using `store_id`.
2. The resulting dataset is joined with Customer Sales using `product_id`.
3. Required business columns are selected.
4. The final dataset is written in Delta format using overwrite mode.

**Gold Output Path**

```text
gold/
└── sales_view/
    └── StoreProductSalesAnalysis/
```

## Data Profiling and Quality Checks

Data profiling was used to understand the structure, completeness, consistency, and reliability of the source datasets.

The profiling activities included:

- Record count and column inspection.
- Schema and data type inspection.
- NULL and blank value examination.
- Duplicate record inspection.
- Date format validation.
- Categorical value inspection.
- Relationship validation between Product and Store.

**Data Quality Observation**

During Product–Store integration, the Store IDs referenced in the Product dataset did not match the available Store IDs in the Store dataset.

A LEFT JOIN was implemented to preserve the Sales records while retaining unmatched Store information as NULL.

This issue was documented rather than introducing artificial relationships.

## Security and Access Management

Secure access to ADLS Gen2 was configured using Azure managed identities and Unity Catalog.

- Azure Data Factory system-assigned managed identity.
- Azure Databricks Access Connector.
- Storage Blob Data Contributor role assignments.
- Unity Catalog storage credential.
- External locations for Bronze, Silver, and Gold containers.

## Final Outcome

The project demonstrates an end-to-end Azure data engineering workflow:

- Automated ingestion using Azure Data Factory.
- Raw data storage using ADLS Gen2.
- Data profiling and transformations using PySpark.
- Delta Lake upsert operations.
- Integration of multiple business datasets.
- Gold-layer Delta table creation.
- Secure cloud storage access.

## Key Learning Outcomes

- Azure Data Factory pipeline development.
- Dynamic file selection using metadata.
- ADLS Gen2 storage management.
- PySpark data transformations.
- Data profiling and quality assessment.
- Delta Lake MERGE operations.
- Medallion Architecture.
- Unity Catalog and external locations.
- End-to-end cloud data engineering.

## Author

Shivam Kumar

**Project:** Azure Data Engineering – Sales View Assignment
