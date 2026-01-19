# NYC Taxi Data Pipeline - Medallion Architecture 🚕 🗽

This project consists of an end-to-end data pipeline developed in **Databricks Free Edition**, utilizing the **Medallion Architecture**. The objective is to ingest, process and analyze public New York City taxi data [(TLC Trip Record Data)](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) with a focus on scalability, data quality, and governance through **Unity Catalog**.



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

## 📂 Repository Structure
The folder src/ is organized to facilitate code reuse:
1. `initial_setup`: Creation of catalogs, schemas, and volumes.
2. `ingestion`: TLC Trip Record Data ingestion and writing in the raw layer.
3. `bronze`: Normalization logic.
4. `silver`: Transformation and Data Quality.
5. `gold`: Unification of taxi data. (Aggregations and business metrics are consolidated on analysis/ folder)
6. `main_pipeline`: Central orchestrator that executes the complete flow.

The folder analysis/ has one notebook with business metrics.

The requirements.txt file was generated with the following command on Databricks:
````
%sh
pip freeze
````



## 🚀 How to Run
1. Clone this repository into your Databricks Workspace using **Repos**.
2. Run the `main_pipeline` notebook to process all layers automatically.
3. Run the analysis/business_analysis notebook in order to see business metrics.



## 🚀 Future Improvements (Roadmap)

To evolve this project into a production-ready data platform, the following steps are planned, categorized by engineering domain:

### 📂 Data Engineering & Orchestration
* **Incremental Ingestion & Transformation:** Implement Delta Lake's `MERGE` logic or Change Data Feed (CDF) to process only new or updated records instead of full overwrites.
* **Cloud Agnostic Storage:** Migrate the raw layer to a cloud-native object store (e.g., AWS S3) to decouple storage from the compute workspace.
* **Geospatial Enrichment:** Perform Joins with the NYC Taxi Zones lookup table to transform location IDs into human-readable Borough and Zone names.

### 🏗️ DevOps & CI/CD
* **Branch Protection:** Implement Branch Protection Rules on the `main` branch to ensure code quality via Pull Requests.
* **CI/CD Pipelines:** Setup GitHub Actions to automate notebook deployment and run unit tests on Spark logic.
* **Tool Agnostic Transition:** Refactor the core logic into a Python package (using Poetry or Setuptools) to allow the pipeline to run on any Spark-compliant environment (AWS EMR, GCP Dataproc, or local clusters).

### 🛡️ Data Governance & Quality
* **Data Quality Testing:** Integrate a validation framework to enforce "hard" constraints on the Silver layer (e.g., ensuring `total_amount` is never null or negative) before writing to the table.
* **Monitoring & Alerting:** Implement an automated monitoring system to send notifications (Slack/Email) if the ingestion volume drops below a specific threshold or if a job fails.

### 📝 Knowledge Sharing
* **Medium Article:** Write a technical post detailing the challenges of handling NYC Taxi schema evolution and the benefits of the Medallion Architecture.
* **Documentation Expansion:** Create a detailed Data Dictionary in the Unity Catalog for all Gold layer assets.
