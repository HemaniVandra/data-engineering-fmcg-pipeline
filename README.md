# 🏭 Data Engineering FMCG Pipeline — Databricks Lakehouse

> An end-to-end ETL pipeline built on **Databricks** for a real-world FMCG industry use case — consolidating data from two companies into a unified **Medallion Lakehouse Architecture**.

---

## 📌 Project Overview

In this project, a large retail company acquires a smaller one. The parent company's data pipeline is already operational. This project focuses on **architecting and building the complete data pipeline for the child company** — ingesting, transforming, and consolidating its data into the same lakehouse, enabling unified reporting and analytics across both entities.

This project is suitable for **beginners and advanced data engineers** alike, built entirely on the **Databricks Free Edition**.

---

## 🛠️ Tech Stack

| Tool / Technology | Purpose |
|---|---|
| **Python** | Core scripting and data processing |
| **SQL** | Transformations and querying |
| **Apache Spark** | Distributed data processing |
| **Databricks (Free Edition)** | Unified analytics platform |
| **Amazon S3** | Cloud storage / data lake |
| **Medallion Architecture** | Bronze → Silver → Gold data layers |
| **BI Dashboard & Genie** | Reporting and analytics layer |

---

## 🏗️ Architecture

The pipeline handles **two distinct flows** for the child company: a one-time historical full load and a recurring incremental load. Both flows ultimately merge into the parent company's Gold layer.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SOURCE SYSTEMS                               │
│  OLTP Server (historical)              Incremental source (daily)   │
└────────────┬────────────────────────────────────┬───────────────────┘
             │                                    │
             ▼                                    ▼
┌────────────────────────┐          ┌─────────────────┐   ┌───────────────────┐
│   S3 — Historical data │          │  S3 / landing   │──▶│  S3 / processed   │
│   (dim & fact tables)  │          │  (new orders)   │   │  (archived files) │
└────────────┬───────────┘          └────────┬────────┘   └───────────────────┘
             │                               │ (files moved after load)
             ▼                               ▼
┌────────────────────────┐          ┌─────────────────┐
│   Bronze — historical  │◀─ merge ─│  Bronze staging │
│   (raw child co. data) │          │  (incremental)  │
└────────────┬───────────┘          └────────┬────────┘
             │                               │
             ▼                               ▼ (transform & clean)
┌────────────────────────┐          ┌─────────────────┐
│   Silver — historical  │◀─ merge ─│  Silver staging │
│   (cleaned child data) │          │  (incremental)  │
└────────────┬───────────┘          └────────┬────────┘
             │                               │ (upsert)
             └──────────────┬────────────────┘
                            ▼
              ┌─────────────────────────┐
              │  Gold — child company   │
              │  (unified child data)   │
              └────────────┬────────────┘
                           │ merge
                           ▼
              ┌─────────────────────────┐
              │  Gold — parent + child  │  ◀── Parent company (pre-existing)
              │  (unified FMCG data)    │
              └────────────┬────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │   BI Dashboard & Genie  │
              │   (business insights)   │
              └─────────────────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │   Drop staging tables   │
              │   (pipeline cleanup)    │
              └─────────────────────────┘
```

---

## 🔄 Pipeline Flows

### Flow 1 — Historical Full Load
This is a one-time load of all historical data for the child company.

1. Raw data is exported from the child company's **OLTP server** to **Amazon S3**
2. Data is ingested into the **Bronze historical layer** (raw, as-is)
3. Data is cleaned and standardized into the **Silver historical layer**
4. Transformed data is loaded into the **Gold child company layer**
5. Gold child data is **merged into the Gold parent layer** to form the consolidated lakehouse

### Flow 2 — Incremental Load (Daily Orders)
This runs on a recurring schedule to process new daily order data.

1. New order files land in **S3 / landing** folder
2. Files are loaded into **Bronze staging layer**
3. Source files are **moved from `landing` to `processed`** in S3 (ensuring no double processing)
4. Bronze staging is **merged into Bronze historical layer**
5. Data is transferred to **Silver staging** where all transformations and cleaning occur
6. Cleaned silver staging data is **merged into Silver historical layer** (child company)
7. Silver staging data is **upserted into the Gold child company layer**
8. Gold child data is **merged into the Gold parent layer**
9. **Staging tables (Bronze & Silver) are dropped** to keep the environment clean

---

## 📂 Project Structure

```
data-engineering-fmcg-pipeline/
│
├── README.md
│
├── jobs/
│   └── fmcg_incremental_pipeline_config.json   ← Databricks job config (importable)
│
├── 0_data/
│   ├── 1_parent_company/
│   │   ├── full_load/
│   │   │   ├── dim_customers.csv
│   │   │   ├── dim_gross_price.csv
│   │   │   ├── dim_products.csv
│   │   │   └── fact_orders.csv
│   │   └── incremental_load/
│   │       ├── fact_orders.csv
│   │       └── incremental_data_parent_company_query.dbquery
│   │
│   └── 2_child_company/
│       ├── full_load/
│       │   ├── customers/
│       │   ├── gross_price/
│       │   ├── orders/landing/     ← Per-day historical order CSV files
│       │   └── products/
│       └── incremental_load/
│           └── orders/             ← Per-day incremental order files
│
└── 1_codes/
    ├── 1_setup/
    │   ├── dim_date_table_creation.ipynb
    │   ├── setup_catalog.ipynb
    │   └── utilities.ipynb
    ├── 2_dimension_data_processing/
    │   ├── 1_customers_data_processing.ipynb
    │   ├── 2_products_data_processing.ipynb
    │   └── 3_pricing_data_processing.ipynb
    └── 3_fact_data_processing/
        ├── 1_full_load_fact.ipynb
        └── 2_incremental_load_fact.ipynb
```

---

## 📊 Dataset

> ⚠️ Sample/anonymized data is provided for demonstration purposes only. No real customer or business data is included.

| Table | Description |
|---|---|
| `dim_customers` | Customer master data |
| `dim_products` | Product catalog |
| `dim_gross_price` | Pricing data across product lines |
| `fact_orders` | Order transactions (full + incremental loads) |

---

## 🚀 Getting Started

> **Note:** This project is built on **Databricks** and uses Databricks-specific features like `%fs`, `%sql`, and `dbutils`. A Databricks workspace is required to run the notebooks.

### Prerequisites
- Databricks Free Edition account → [Sign up here](https://www.databricks.com/try-databricks)
- Amazon S3 bucket (or Databricks-managed storage)
- Basic knowledge of Python and SQL

### Steps to Run

**Historical load:**
1. Clone this repository
2. Upload notebooks from `1_codes/` to your Databricks workspace
3. Upload data files from `0_data/` to your S3 bucket
4. Run in order: `1_setup/` → `2_dimension_data_processing/` → `3_fact_data_processing/1_full_load_fact.ipynb`

**Incremental load:**
1. Place new daily order files in the `S3/landing/` folder
2. Run `3_fact_data_processing/2_incremental_load_fact.ipynb`
3. The pipeline will automatically move files to `S3/processed/`, run all transformations, upsert into Gold, and clean up staging tables

---

## ⚙️ Pipeline Orchestration

The incremental load is fully orchestrated using a **Databricks Job** named `FMCG Incremental Update`. The job is triggered automatically as soon as a new file arrives in the S3 landing bucket — no manual intervention needed.

The job contains 4 notebooks executed in a sequential dependency chain:

```
[1] dim_processing_customer
    └── source: 1_codes/2_dimension_data_processing/1_customers_data_processing.ipynb
         │
         ▼
[2] dim_processing_products
    └── source: 1_codes/2_dimension_data_processing/2_products_data_processing.ipynb
         │
         ▼
[3] dim_processing_pricing
    └── source: 1_codes/2_dimension_data_processing/3_pricing_data_processing.ipynb
         │
         ▼
[4] fact_processing_orders
    └── source: 1_codes/3_fact_data_processing/2_incremental_load_fact.ipynb
```

Each notebook task only starts after the previous one completes successfully, ensuring data integrity across dimension and fact processing steps.

### Importing the Job into Your Workspace

The full job configuration is exported and saved at `jobs/fmcg_incremental_pipeline_config.json`. To reuse it in your own Databricks workspace:

1. Go to **Workflows** in your Databricks workspace
2. Click **Create Job** → **Import**
3. Upload `jobs/fmcg_incremental_pipeline_config.json`
4. Update the notebook paths and S3 trigger settings to match your environment

---

## 💡 Key Concepts Demonstrated

- Full Load vs Incremental Load patterns
- Staging layer design for incremental pipelines
- S3 landing-to-processed file management pattern
- Medallion Architecture (Bronze / Silver / Gold)
- Multi-source data consolidation (parent + child company)
- Merge and Upsert operations in Delta Lake
- Spark-based distributed data processing
- Pipeline cleanup and staging table lifecycle management
- Databricks Jobs orchestration with task dependencies
- Event-driven pipeline triggering (S3 file arrival)

---

## 🗺️ Roadmap

| Feature | Status |
|---|---|
| Historical full load pipeline | ✅ Done |
| Incremental load pipeline | ✅ Done |
| S3 event-driven job trigger | ✅ Done |
| Databricks Jobs orchestration | ✅ Done |
| BI Dashboard & Genie | ✅ Done |
| Unity Catalog integration | 🔜 Planned for v2 |

> **Note:** Unity Catalog is not implemented in the current version. It is planned as a key feature in the next iteration of this project.
