# Sports Equipment Sales — End-to-End Data Engineering Project

## 1. Project Overview
This project processes sports equipment sales transactions to analyze revenue by product category, region, and payment method.

It uses Databricks, PySpark, Delta Lake, and Medallion Architecture to transform raw sales data into clean, structured datasets for business reporting.

The pipeline supports full loads, incremental processing, SCD Type 1 updates, schema evolution, watermark management, and workflow orchestration using Databricks Lakeflow Jobs.

## 2. Technology Stack
- Databricks — Data processing and orchestration
- PySpark — Data transformation and validation
- Delta Lake — Managed Delta tables, MERGE, and version history
- Unity Catalog — Catalog, schema, and table organization
- Lakeflow Jobs — Full-load and incremental workflow orchestration
- Git and GitHub — Code version control and collaboration

## 3. Data Architecture
The project follows Medallion Architecture:

**Source → Bronze → Silver → Gold**

- **Source:** Stores incoming sales CSV files.
- **Bronze:** Contains ingested source data.
- **Silver:** Contains validated, cleaned, and deduplicated sales records.
- **Gold:** Contains Dimension tables, the Sales Fact table, and business KPI tables.

## 4. Gold Dimensional Model
Four Dimension tables:
- `dim_customers`
- `dim_products`
- `dim_store`
- `dim_salesperson`

Fact table:
- `fact_sales`

**Fact grain:** One record per `sale_id`.

Dimension records are maintained using SCD Type 1 logic, and the Fact table references dimensions through surrogate keys.

## 5. Incremental Processing
- Uses watermark-based filtering to identify newer records.
- Uses Delta MERGE to insert new records and update existing records.
- Handles duplicate sales using the latest `last_updated` timestamp.
- Supports schema evolution, including the `sales_channel` and `loyalty_tier` columns.
- Advances the watermark after successful Silver, Gold, and KPI processing.

## 6. Databricks Notebooks
| Notebook | Responsibility |
|---|---|
| `full_load` | Source ingestion and Bronze/Silver full load |
| `gold_full_load` | Create Gold Dimensions and Fact |
| `incremental_merge` | Process incremental data into Silver |
| `gold_incremental_merge` | Update Gold Dimensions and Fact |
| `gold_kpi` | Refresh business KPI tables |
| `watermark_initialize` | Initialize the watermark control table |
| `watermark_commit` | Commit the processed watermark |

## 7. Lakeflow Jobs

**Full Load Job:**

`full_load → gold_full_load → gold_kpi`

**Incremental Job:**

`incremental_merge → gold_incremental_merge → gold_kpi → watermark_commit`

The initial watermark was configured separately using `watermark_initialize`.

## 8. Business Analysis
The Gold layer supports:
- Sales revenue analysis by product category
- Revenue analysis by store region
- Completed-sales revenue
- Sales analysis by payment method

## 9. Verified Results — After Batch 03
| Table | Records |
|---|---:|
| Silver | 35,480 |
| Fact Sales | 35,480 |
| Customers Dimension | 2,600 |
| Products Dimension | 225 |
| Store Dimension | 64 |
| Salesperson Dimension | 136 |

**Data-quality validation:**
- Fact table unique sale IDs: 35,480
- Fact table NULL sale IDs: 0
- Dimension business keys: Unique and non-NULL

**Final committed watermark:** `2026-09-27 10:00:00`

## 10. GitHub Integration
The completed Databricks notebooks are maintained in this GitHub repository using Databricks Git folders and feature-branch version control.