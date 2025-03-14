# Data-Engineer-Project-Cloud-Azure-Services-Car-Catalog

## Overview
This project is a hands-on guide to building an end-to-end Azure Data Engineering solution. Using real-world sales transaction data from Tableau Public Datasets, it covers essential tools, techniques, and best practices for creating scalable and efficient data pipelines. 

![Image](https://github.com/user-attachments/assets/b8201839-8dce-4be5-b0d5-d11fe30c1e7c)


## Architecture
The project follows the Medallion Architecture:
* Bronze Layer: Raw data ingestion from GitHub to Azure Data Lake Storage (ADLS) using ADF.
* Silver Layer: Data cleaning and transformation using Databricks Auto-loader & PySpark.
* Gold Layer: Optimized storage using Delta Live Tables (DLT).


## Purposes
* Learn Azure Data Services – Get familiar with tools like Azure Data Factory, Azure Databricks, Azure Storage Account, and Azure SQL Server.
* Build End-to-End Data Pipelines – Practice creating and managing ETL/ELT workflows in Azure.
* Transform and Process Data – Clean, transform, and aggregate data using Azure Data Flow and SQL.
* Implement Slow Changing Dimensions (SCD): Learn how to manage historical data changes by implementing SCD Type 1 (Upsert) technique.
* Surrogate Key Usage: Understand how to use surrogate keys to maintain unique records and efficiently manage dimensional data in your data warehouse.
* Automate with Databricks Workflows: Get familiar with Databricks notebooks and workflows to automate data processing, transformations, and orchestration.

## Tech Stack
Data Sources & Ingestion
* SQL Database – Source system for structured data storage.
* Azure Data Factory (ADF) – ETL/ELT tool for ingesting data from SQL to Azure Data Lake.
* GitHub – Source of the raw dataset.

Data Storage
* Azure Data Lake Storage Gen2 (ADLS Gen2) – Cloud-based data lake for raw, transformed, and serving layers.
* Parquet Format – Columnar storage format used for raw and transformed data.
* Delta Lake – Storage layer for serving data, enabling ACID transactions and time travel.

Data Processing & Transformation
* Databricks (Apache Spark-based processing) – Used for transforming raw data into structured formats.
* Databricks Workflows – Orchestrating and automating multiple notebooks for pipeline execution.
* Incremental Data Processing – Handling real-time and batch data loads.
* One Big Table (OBT) & Star Schema – Different data modeling techniques for analytics.
* Slow Changing Dimensions (SCD) & Surrogate Keys – Implemented for historical tracking and efficient lookups.

Data Modeling
* Star Schema – Dimensional modeling approach for reporting and analytics.

