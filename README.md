# NYC Taxi Data Pipeline - Medallion Architecture 🚕 🗽

This project consists of an end-to-end data pipeline developed in **Databricks Free Edition**, utilizing the **Medallion Architecture**. The objective is to ingest, process and analyze public New York City taxi data (TLC Trip Record Data) with a focus on scalability, data quality, and governance through **Unity Catalog**.



## 🏗️ Solution Architecture

The pipeline was designed following Data Engineering best practices, ensuring modularity and decoupling between layers.

### Project Layers:
* **Raw (Volumes):** Storage of original Parquet files within **Unity Catalog Volumes**.
* **Bronze:** Ingestion of raw data with schema normalization. Here, I resolved challenges related to column name inconsistencies (case-sensitivity) and divergent primitive types across different months.
* **Silver:** Cleaning and refinement. Application of sanity filters (positive fares/distances) and temporal filters to remove outliers caused by GPS sensor errors.
* **Gold:** Analytics layer. Consolidation of different fleets (Yellow + Green)
* **Business Analysis:** Creation of aggregated views to answer business questions.

## 🛠️ Technologies and Tools
* **Databricks Free Edition:** Development and Spark execution environment.
* **PySpark:** Distributed processing engine.
* **Delta Lake:** Storage format to ensure ACID transactions and Schema Evolution.
* **Unity Catalog:** Governance and management of permissions and Volumes.
* **Orchestration:** Centralized via `dbutils.notebook.run`, allowing modular execution of the entire workflow.

## 🧩 Key Technical Challenges Overcome

### 1. Schema Inconsistency (Schema Evolution)
NYC Taxi Parquet files frequently undergo schema changes (e.g., uppercase vs. lowercase columns, different data types) between months. I implemented a generic standardization function that uses mapping dictionaries and the Spark SQL engine to resolve column names in a case-insensitive manner, preventing pipeline failures.

### 2. Partitioning and Performance
The final tables were partitioned by **Year** and **Month**. This optimizes Delta Lake's *Data Skipping* capabilities, ensuring that analytical queries on specific months do not need to scan the entire historical database.

## 📂 Notebook Structure
The repository is organized to facilitate code reuse:
1. `initial_setup`: Creation of catalogs, schemas, and volumes.
2. `ingestion`: TLC Trip Record Data ingestion and writing in the raw layer.
3. `bronze`: Normalization logic.
4. `silver`: Transformation and Data Quality.
5. `gold`: Unification of taxi data. (Aggregations and business metrics are consolidated on analysis/ folder)
6. `main_pipeline`: Central orchestrator that executes the complete flow.



## 🚀 How to Run
1. Clone this repository into your Databricks Workspace using **Repos**.
2. Run the `main_pipeline` notebook to process all layers automatically.
3. Run the analysis/business_analysis notebook in order to see business metrics.
