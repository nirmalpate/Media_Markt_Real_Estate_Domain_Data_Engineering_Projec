# Media_Markt_Real_Estate_Domain_Data_Engineering_Projec

## 💾 Media Markt Analytics Data Pipeline

This repository documents the setup and artifacts for a cloud-based analytical pipeline focused on sales and operational data for Media Markt. The architecture utilizes **Snowflake** for scalable data warehousing, **Snowpipe** for continuous ingestion, **Snowflake Tasks** for automated transformations, and **Power BI** for reporting.

### ✨ Key Features

  * **Cloud Data Warehouse:** Centralized and scalable data processing powered by **Snowflake**.
  * **Automated Ingestion (Snowpipe):** Near real-time loading of raw files from cloud storage (e.g., S3/Azure/GCS) into Snowflake.
  * **Automated ELT:** Scheduled data transformations and dimensional model builds using **Snowflake Tasks** and **Streams**.
  * **Dimensional Modeling:** Optimized Star Schema design for efficient analytics and fast query performance.
  * **Business Intelligence:** Interactive sales, inventory, and fulfillment reporting using the provided **Power BI** file.

### 🏗️ Architecture Flow

1.  **Source:** Raw data files (e.g., CSV, JSON) land in a designated cloud storage location.
2.  **Ingestion:** **Snowpipe** is configured to monitor the cloud storage stage and automatically load new files into the raw staging tables in Snowflake.
3.  **Transformation:** Scheduled **Snowflake Tasks** execute data cleaning, key generation, and dimensional model loading (ELT) from the staging area into the final **Fact** and **Dimension** tables.
4.  **Reporting:** **Power BI** connects to the clean, dimensional tables in Snowflake, offering a single source of truth for reporting and dashboards.

### 📦 Repository Contents

| Directory/File | Description |
| :--- | :--- |
| `sql/` | Directory containing all SQL scripts for setup, including schema creation, table definitions, Snowpipe setup, and Task/Stream logic. |
| `sql/schema_setup.sql` | SQL to create the necessary databases, schemas (`STAGING`, `ANALYTICS`), and warehouse configuration. |
| `sql/dimensional_model.sql` | SQL for defining **DIMENSION** and **FACT** tables in the `ANALYTICS` schema. |
| `sql/snowpipe_tasks.sql` | SQL to create the External Stages, Snowpipes, Streams, and scheduled Tasks. |
| `powerbi/` | Contains the finished Power BI report file. |
| `powerbi/Media_Markt.pbix` | The final Power BI report, including the data model and visualizations. |
| `README.md` | This documentation file. |

### 🛠️ Prerequisites

  * **Snowflake Account:** With `ACCOUNTADMIN` or sufficient roles to create warehouses, databases, integrations, pipes, and tasks.
  * **Cloud Storage:** Access to S3, Azure Blob, or GCS for staging raw data.
  * **SnowSQL CLI:** Or any preferred SQL development tool (DBeaver, VS Code extension, etc.).
  * **Power BI Desktop:** To open and publish the reporting dashboard.

### 🚀 Deployment Instructions

#### 1\. Snowflake Environment Setup

Execute the schema and table creation scripts:

```bash
# Execute initial setup
snowsql -f sql/schema_setup.sql
snowsql -f sql/dimensional_model.sql
```

#### 2\. Configure Snowpipe for Ingestion

1.  **Create Storage Integration:** Grant Snowflake access to your cloud storage.
2.  **Create External Stage:** Define the location where raw files land.
3.  **Define Snowpipe:** Create the pipe object, linking the stage to your raw staging table.
    ```sql
    -- Example from sql/snowpipe_tasks.sql
    CREATE PIPE STAGING.SALES_PIPE AUTO_INGEST=TRUE AS
    COPY INTO STAGING.RAW_SALES
    FROM @MEDIA_MARKT_STAGE
    FILE_FORMAT = (TYPE = CSV ...);
    ```
4.  **Connect Notification:** Configure the cloud storage to send events (e.g., file upload notification) to the Snowpipe endpoint for automated, continuous loading.

#### 3\. Set up Automated Transformations

1.  **Create Streams:** Use Streams on your raw staging tables to track new/changed data efficiently.
2.  **Create Stored Procedures:** Write the ELT logic (data validation, lookups, dimension key generation, fact loading) into Stored Procedures.
3.  **Define Tasks:** Create the root Task, scheduled to run periodically, that calls the Stored Procedures.
    ```sql
    -- Example from sql/snowpipe_tasks.sql
    CREATE TASK ANALYTICS.LOAD_FACT_SALES
    WAREHOUSE = REPORTING_WH
    SCHEDULE = '15 MINUTE'
    WHEN SYSTEM$STREAM_HAS_DATA('STAGING.RAW_SALES_STREAM')
    AS CALL ANALYTICS.SP_LOAD_FACT_SALES();
    ```
4.  **Activate Tasks:**
    ```sql
    ALTER TASK ANALYTICS.LOAD_FACT_SALES RESUME;
    ```

#### 4\. Power BI Reporting

1.  **Open Report:** Open the `Media_Markt.pbix` file.
2.  **Connect Data:** Update the Power BI data source connection details to your Snowflake user and connection information.
3.  **Visualize and Analyze:** The report provides pre-built dashboards for sales performance, inventory levels, and fulfillment metrics.
4.  **Publish:** Publish the report to the Power BI Service for sharing and set up a Power BI Gateway for scheduled data refreshes from Snowflake.
