# Netflix Azure Data Engineering Lakehouse

## Project Overview

This project demonstrates the development of an Azure-based data engineering pipeline for processing Netflix datasets using a Medallion Architecture.

The solution uses Azure Data Factory (ADF), Azure Data Lake Storage Gen2 (ADLS Gen2), Azure Databricks, PySpark, and Delta Lake to ingest, validate, transform, and prepare data for analytics.

The project focuses on building a scalable, maintainable pipeline with data quality checks, quarantine handling, and incremental processing.

**Project Status:** In development. Bronze ingestion and Silver data quality preparation are underway/completed in stages; incremental Silver processing and Gold analytics are not yet confirmed as implemented.

## Azure Infrastructure

The project uses the following Azure services:

| Azure Service | Purpose |
|---|---|
| Azure Data Factory | Orchestrates data ingestion and pipeline execution |
| Azure Data Lake Storage Gen2 | Stores datasets across Raw, Bronze, Silver, and Gold layers |
| Azure Databricks | Executes PySpark-based ingestion and data transformations |
| Delta Lake | Provides transactional storage and supports incremental processing |

## Storage Architecture

The Azure storage account is organized into four containers:

| Container | Purpose |
|---|---|
| `raw` | Stores the original Netflix CSV datasets |
| `bronze` | Stores ingested data in Delta format, including ingestion metadata |
| `silver` | Intended for validated, cleaned, and standardized Delta datasets |
| `gold` | Intended for analytics-ready datasets |

This separation follows the Medallion Architecture, allowing raw data to be preserved while progressively improving data quality.

## Source Datasets

The pipeline processes five Netflix datasets:

1. `netflix_titles`
2. `netflix_cast`
3. `netflix_directors`
4. `netflix_countries`
5. `netflix_category`

These datasets provide information about Netflix titles, cast members, directors, countries, and content categories.
