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

## Implementation Steps
### Step 1: Setting Up Azure Resources
  1. Create an **Azure Resource Group.**
  2. Create **Storage account**
      - Performance - Standard: Recommended for most scenarios (general-purpose v2 account)
      - Redundancy - Locally-redundant storage (LRS)
      - Enable hierarchical namespace - Enabled
      - Access tier - Hot: Optimized for frequently accessed data and everyday usage scenarios
      - Create Containers in Data storage
        - bronze
        - silver
        - gold
  3. Create **Azure Data Factory (ADF)**
  4. Create **Azure SQL**
      - Create **SQL databases (Single database)**
         - Create **SQL Database Server**
            - Authentication method - Use both SQL and Microsoft Entra authentication
            - Set Microsoft Entra Admin - My User
            - Workload environment - Development
            - Compute + Storage
               - Networking
                 - Connectivity method - Public endpoint
                 - 
  5. Create **source_car_data** in **SQL Database**
       ```sql
       CREATE TABLE source_car_data (
        	branch_id VARCHAR(200),
        	dealer_id VARCHAR(200),
        	model_id VARCHAR(200),
        	revenue BIGINT,
        	unit_sold BIGINT,
        	date_id VARCHAR(200),
        	day INT,
        	month INT,
        	year INT,
        	branch_name VARCHAR(200),
        	dealer_name VARCHAR(200),
        	product_name VARCHAR(200)
        )
  6. Set up Azure Databricks workspace and configure clusters.
     - Create a Unity Metastore
       - In https://accounts.azuredatabricks.net/
         - User management
            - Assign a role **Account admin** to my account
         - Catalog
            - Create metastore
              - ADLS Gen 2 path
                 - Create **Access Connector for Azure Databricks**
                 - Go to **Storage Account**
                   - Go to **Access Control (IAM)**
                     - Click on **+ADD** and **Add role assignment
                     - Search for **Storage Blob Data Contributor**
                        - Assign access to - Managed Identity
                        - Members - Click on **+Select members** and Choose **Access Connector** that we created
                 - Create a container in **Storage Account** may be naming as "unitymetastore**
                 - In **ADLS Gen 2 path** add value as **<container_name>@<storage_account_name>.dfs.core.windows.net/<path>**
                 - In **Access Control Id** just copy **Resouce ID** in Access Connector and paste value
                 - Go back to **Catalog**
                   - Click on **Assign to workspace**
                   - Choose metastore that we just created, Click on Assign and then Click Enable
      - Create Compute
          - Click on Create compute
          - Policy - Personal Compute
          - Databricks runtime version - Runtime: 15.4 LTS (Scala 2.12, Spark 3.5.0)
          - Terminate after 30 minutes of inactivity
      - Create External Data Location
        - Go to Catalog
          - Go to Credentials
          - Click on Create Credential
          - In **Access Control Id** just copy **Resouce ID** in Access Connector and paste value
        - Click on Create external location
          - URL - add value as **abfss://<container_name>@<storage_account_name>.dfs.core.windows.net/<path>**
          - Storage credential - Choose our Managed Identity
        - Go back to https://accounts.azuredatabricks.net/
          - Go to Catalog
          - Edit our account & Choose our normal account (account without #ext#)
        - Click **Test Connection** in Catalog
        - Repeat the step with Silver and Gold containers
       
### Step 2: Data Ingestion Using Azure Data Factory (ADF)   
  1. Create Linked Services for GitHub (HTTP connector) and Azure SQL Database.
     - Go to **Manage** tab
       - In **Connections** menu, click **Linked services** and click **+ New**
         - Search for **HTTP**
           - In Base URL - You can find data in this link https://github.com/anshlambagit/Azure-DE-Project-Resources/tree/main/Raw%20Data
           - Click on Create
         - Search for **Azure SQL Database**
           - Go to **Azure SQL Database**
             - Go to Security > Networking
               - In **Exceptions** check Allow Azure services and resources to access this server
              
  2. Create **ingestion_source_data** pipeline
     - In **Activities** tab
       - search for **Copy data** and drag into canva
     - In **Source** menu
       - Click on +New
       - Seach for HTTP
       - Choose CSV format
       - In Relative URL Paste this URL: https://github.com/anshlambagit/Azure-DE-Project-Resources/blob/main/Raw%20Data/SalesData.csv
       - Click on Advanced
         - In Parameters menu
           - Name "load_flag"
         - In Relative URL
           - Click Add dynamic content and Add value like this ```anshlambagit/Azure-DE-Project-Resources/refs/heads/main/Raw%20Data/@{dataset().load_flag}```
     - Go to our Pipeline
       - Click on **Sink** menu
         - In Sink dataset, seach for **Azure SQL Database**
       - Click on **Mapping** menu
         - Setup mapping columns and data type between source and table
     - Click on Debug button
    
  3. Create **incremental_data** pipeline
      - In Activities tab search for Copy data and drag into canva
      - Go to SQL Database
        - Create **Watermark_table**
          ```sql
          CREATE TABLE watermark_table (
	            last_load VARCHAR(2000)
              )
      - INSERT value which is initial date_id (based on this project) to **Watermark_table**
        ```sql
          INSERT INTO watermark_table
            VALUES ('DT00001')
      - Create PROCEDURE "update_watermark_table"
        ```sql
          CREATE PROCEDURE update_watermark_table
          	@lastload VARCHAR(200)
          AS
          BEGIN
          	-- Start the transaction
          	BEGIN TRANSACTION;
          
          	-- Update the incremental column in the table
          	UPDATE watermark_table
          	SET last_load = @lastload
          	COMMIT TRANSACTION;
          	END;
      - In Activities tab search for **Lookup** and drag into canva
        - last_load
          - In source dataset choose **SQL Database**
          - Parameters - table_name
          - In connection
            - Table choose **Enter manually**
            - Table name Click Add dynamic content
            - In Dataset properties
              - Parameter table_name add value "watermark_table
      	  - In setting menu
            - Use query - Query
              -  ```sql
                 SELECT
                    *
                 FROM
                    watermark_table
	 	- current_load
   		  - In source dataset choose **SQL Database**
       		  - First row only - Checkout
                  - Use query - Query
                  	```sql
	                 	SELECT
	                    		MAX(date_id) AS max_date
	                 	FROM
	                    		source_cars_data
			- Click Debug
       	- Copy_Incremental_Data
          - In source dataset choose **SQL Database**
          - Use query - Query
             ```sql
                SELECT
    		  *
                FROM
                  source_car_data
                WHERE
                     date_id >  '@{activity('last_load').output.value[0].last_load}'
                 AND date_id <= '@{activity('current_load').output.value[0].max_date}'
           - In Sink menu
             - Sink Dataset with **Azure DataLake Gen2**
             - Choose Parquet format
             - File path = path folder that we created such as ```bronze/rawdata/<filename>```
             - Import schema - None
           - In Mapping menu
             - Click on "Import schemas"
               - in ```@activity('last_load').out paste value = MIN(date_id)```  -- In this case is from watermark_table
               - in ```@activity('current_load').out paste value = MAX(date_id)``` -- In this case is from source_cars_data
              
      - Create Store Procedure
        - In Activities tab search for **Stored procedure** and drag into canva
        - In setting menu
          - Linked Service with SQL Database
          - Stored procedure name in this case is StoreProcedure = update_watermark_table
          - Stored procedure parameters
            - last_load | value = ```@activity('current_load').output.value[0].max_date```
        - Click Debug
               
   	   

	  	
		

                      
	  
 	
       

 
      











    
