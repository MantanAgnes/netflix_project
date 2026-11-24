#### Netflix End-to-End Azure Data Engineering Lakehouse

An end-to-end, incremental **Medallion Architecture** built on **Azure Data Factory**, **Azure Databricks**, **Delta Lake**, **Unity Catalog**, and **Azure Data Lake Storage Gen2 (ADLS Gen2)**. This project orchestrates raw multi-file ingestion into a governed **Kimball Dimensional Star Schema**, powering analytical SQL views and a **Databricks AI/BI Dashboard with Genie**. 

#### Architecture Overview


[GitHub Raw CSVs: 4 Lookup Files]      [Local CSV: netflix_titles]
       │  (Azure Data Factory Pipelines)            |  (Direct Upload)
       ▼                                            
[ADLS Gen2: raw/ (Unified Landing Zone for all 5 Datasets)]
       │
       │  (Databricks Auto Loader + Checkpoints)
       ▼
[Bronze Layer: bronze/] ──────────► Raw Delta tables + _rescued_data & audit metadata
       │
       │  (Structured Streaming availableNow=True + Quality Gates)
       ▼
[Silver Layer: net_schema] ──────► Cleaned, deduplicated Delta tables
       │                             └─► [_quarantine/] Rejected & orphan rows with failure reasons
       │
       │  (Kimball Dimensional Modelling + Surrogate Keys)
       ▼
[Gold Layer: gold] ──────────────► 11-Table Star Schema (Fact, Dimensions & Bridge Tables)
       │
       ▼
[Analytics & Self-Service BI] ───► 5 Pre-Aggregated SQL Views + Databricks AI/BI Dashboard & Genie

#### Key Engineering Highlights

#### 1. Cloud Ingestion & Orchestration (adf/ & databricks/01_bronze/)

* **Azure Data Factory (ADF):** Automates initial landing of raw multi-entity Netflix CSV files (netflix_titles, netflix_directors, netflix_cast, netflix_countries, netflix_category) into ADLS Gen2 raw/.
* **Databricks Auto Loader (cloudFiles):** Incrementally ingests incoming raw files into Bronze Delta tables with schema inference, _rescued_data capture, and ingestion audit columns (_source_file, _ingested_at).

#### 2. Data Quality, Quarantine & Reusable Workers (databricks/02_silver/)

* **Parameterised Lookup Worker:** Built a single widget-driven PySpark worker (silver_lookup_worker) driven by a lookup array to process all four relational lookup tables in parallel.
* **Referential Integrity & Quarantine Routing:** Validates foreign keys (show_id) against the cleaned netflix_titles parent table, isolating malformed CSV splits, duplicate composite pairs, unparseable dates, and orphan records into _quarantine Delta tables tagged with explicit quarantine_reason diagnostics.
* **Idempotent Incremental Upserts:** Uses Structured Streaming (trigger(availableNow=True)) paired with Delta MERGE on natural and composite keys to process only newly arrived commits.

#### 3. Dimensional Star Schema & Governance (databricks/03_gold/)

* **Kimball Star Schema (netflix_catalog.gold):** Models data into fact_netflix_titles (1 row per title to prevent metric inflation), dim_titles, dim_date, 4 conformed dimensions (dim_directors, dim_cast, dim_countries, dim_categories), and 4 many-to-many bridge tables.
* **Unity Catalog & Quality Gates:** Registers all Silver and Gold Delta paths as external tables with automated primary-key uniqueness assertions.

#### 4. Pipeline Orchestration & BI (images/ & docs/)

* **Databricks Workflows:** Chains Bronze Auto Loader, parent silver_layer_titles, parallel For Each lookup processing, and Gold star-schema refreshes in strict dependency order.

 

* **Databricks AI/BI Dashboard & Genie:** Exposes 5 pre-aggregated SQL views (vw_content_overview, vw_genre_performance, vw_country_catalog, vw_top_talent, vw_monthly_addition_trends) for interactive visualisations and natural-language querying.

 

#### Tech Stack

* **Cloud & Storage:** Microsoft Azure, Azure Data Lake Storage Gen2 (abfss://)
* **Ingestion & Orchestration:** Azure Data Factory (ADF), Databricks Auto Loader, Databricks Workflows
* **Processing & Storage Format:** Azure Databricks, Apache Spark (PySpark), Structured Streaming, Delta Lake (MERGE)
* **Governance & BI:** Unity Catalog, Databricks SQL, AI/BI Dashboards & Genie

#### Repository Structure
├── adf/                        # Azure Data Factory pipeline & linked service JSON definitions
├── databricks/
│   ├── 01_bronze/              # Parameterised Auto Loader incremental ingestion
│   ├── 02_silver/              # Parent titles cleaning & reusable lookup worker with quarantine
│   └── 03_gold/                # 11-table Star Schema builder & 5 analytical SQL views
├── datasets/                   # Sample / reference schema metadata
├── docs/                       # Workflow orchestration notes & data dictionary
├── images/                     # Workflow DAG & AI/BI Dashboard screenshots
└── README.md


