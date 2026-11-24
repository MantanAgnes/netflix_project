#### Netflix End-to-End Azure Data Engineering Lakehouse

An end-to-end, incremental **Medallion Architecture** built on **Azure Data Factory**, **Azure Databricks**, **Delta Lake**, **Unity Catalog**, and **Azure Data Lake Storage Gen2 (ADLS Gen2)**. This project integrates multi-source raw files into a governed **Kimball Dimensional Star Schema**, powering analytical SQL views and a **Databricks AI/BI Dashboard with Genie**. 

#### Architecture Overview

```text
[GitHub Raw CSVs: 4 Lookup Files]           [Local CSV: netflix_titles]
       │  (Azure Data Factory Pipelines)            │  (Direct Upload)
       ▼                                            ▼
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

```

#### Key Engineering Highlights

#### 1. Multi-Source Cloud Ingestion (adf/ & databricks/01_bronze/)

* **Azure Data Factory (adfnetflix-agnes — pipeline1):** Built a parameterised, metadata-driven control flow (p_array) to automate ingesting the four relational lookup files (netflix_directors, netflix_cast, netflix_countries, netflix_category) from GitHub into ADLS Gen2 raw/:
  
	• Metadata Lookup & State Capture: Uses a Web activity (Githubmetadata) and Set variable (Set variable1) to inspect source repository metadata.

	• Pre-Ingestion Validation: Uses a Validation activity (Validationgithub with ds_validation / ds_validation1) to confirm source files exist before triggering data movement.

	• Dynamic Parallel Copy: Iterates over p_array via a ForEach loop executing a parameterised Copy Data activity (ds_github → ds_sink), alongside direct landing of netflix_titles into ADLS Gen2 raw/.

* **Hybrid Raw Ingestion into ADLS Gen2:** Automated ingestion of the four relational lookup files (netflix_directors, netflix_cast, netflix_countries, netflix_category) from GitHub via **Azure Data Factory (ADF)** HTTP pipelines, combined with direct landing of the core netflix_titles dataset into the ADLS Gen2 raw/ container.
* **Databricks Auto Loader (cloudFiles):** Incrementally ingests all five raw datasets into Bronze Delta tables using schema inference, _rescued_data capture, and ingestion audit columns (_source_file, _ingested_at).

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

## Repository Structure

```text
netflix-azure-lakehouse/
├── adf/                    # Azure Data Factory configurations
│   ├── pipeline/           # Parameterised ingestion pipeline
│   ├── dataset/            # Source and destination datasets
│   └── linkedService/      # Connection definitions
├── databricks/
│   ├── 01_bronze/          # Incremental Auto Loader ingestion
│   ├── 02_silver/          # Cleansing, validation and quarantine
│   └── 03_gold/            # Star schema and analytical SQL views
├── datasets/               # Source datasets
├── docs/                   # Technical documentation and data dictionary
├── images/                 # Architecture, workflows and BI screenshots
├── tests/                  # Data quality checks and validation evidence
└── README.md               # Project overview and implementation guide
```
