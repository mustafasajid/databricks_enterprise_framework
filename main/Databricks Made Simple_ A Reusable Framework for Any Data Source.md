# Databricks Made Simple: A Reusable Framework for Any Data Source

Managing data ingestion shouldn’t feel like reinventing the wheel every time a new source comes along. Yet, many teams still find themselves writing similar boilerplate code, tweaking connection strings, and redoing schema setups again and again.

Imagine if you could point to your data source, set a few parameters, and let a pre-built, reusable framework handle the rest  consistently, and across multiple databases.

## **Introduction**

In modern data engineering, speed and flexibility are just as important as accuracy. Teams are expected to onboard new data sources quickly, maintain security standards, and ensure consistent ingestion processes — all while juggling multiple formats, clouds, and databases.

To address these challenges, we’ve built a **highly reusable and configurable Databricks utility** designed to make data ingestion **fast, secure, and cloud-agnostic**. Using Databricks widgets, this framework allows anyone — from seasoned data engineers to analysts — to connect to a new source or target without touching the codebase.

This article walks you through the **idea, design philosophy, and benefits** of this utility before diving into its architecture and usage.

## **Problem Statement**

For many organizations, data ingestion is still a **manual, repetitive, and error-prone process**. Connecting to a new source often means:

* Hardcoding connection details and authentication credentials.

* Duplicating ingestion scripts for each new table or schema.

* Spending hours tweaking paths, formats, and schema mappings.

* Struggling with multi-cloud setups where tools aren’t natively interoperable.

These inefficiencies lead to slower project delivery, inconsistent practices across teams, and increased maintenance overhead.

What’s missing is a **single, flexible framework** that can adapt to different sources, formats, and environments — without requiring developers to rebuild the ingestion logic every time.

## **Solution Overview**

At its core, this Databricks utility acts as a **control center** for managing data connections and orchestrating ingestion workflows — all without manually editing code. Instead, users simply provide parameters through Databricks widgets, and the utility takes care of the rest.

Here’s how it works at a high level:

1. **Parameter Retrieval**  
    The process begins by reading the values you provide through widgets. These values — such as connection details, source formats, and target destinations — become the building blocks for everything that follows.

2. **Connection Metadata Management**  
    The utility checks if the connection already exists in a central metadata table.

   * If it’s already there, the existing details are loaded.

   * If not, the new connection information is added to keep everything tracked in one place.

3. **Metadata Table Initialization**  
    Before ingestion begins, the framework ensures that all required metadata tables exist by calling a supporting setup notebook. This step helps standardize ingestion processes across different projects and teams.

4. **Table Metadata Population**  
    If no table-level metadata exists for the connection, the utility automatically triggers a process to populate it — making sure the pipeline always knows what data it’s working with.

5. **Column Metadata Aggregation**  
    The utility then compiles and summarizes column-level details for each table, providing quick insights such as the number of columns per table. This makes it easier to validate and understand the dataset before ingestion.

6. **Scheduled Ingestion Setup**  
    For ongoing ingestion needs, users can provide a cron expression. The utility will then set up a scheduled job that runs automatically at the defined intervals.

7. **Immediate Ingestion Trigger**  
    If you want results right away, you can kick off ingestion immediately — no waiting for the next scheduled run.

Throughout the process, the utility uses **conditional logic** to avoid unnecessary duplication of metadata or ingestion runs. Whether you’re scheduling regular data refreshes or running a one-off ingestion, the framework adapts to your workflow with minimal configuration.

## **Architecture & Design Decisions**

The Databricks utility is designed around a **metadata-first architecture** — ingestion logic is not hardcoded, but instead driven entirely by structured metadata tables. This ensures consistency, reusability, and flexibility across multiple projects, sources, and targets.

### **1\. Parameter-Driven Execution**

* All runtime parameters are collected through **Databricks widgets**.

* These parameters map directly to fields in the metadata tables, ensuring that ingestion runs are fully documented and reproducible.

### **2\. Metadata-Driven Control Layer**

At the heart of the design are **specialized metadata tables** that store every aspect of the ingestion process:

* **`connection_metadata`**  
   Stores core connection details — type, host, port, database, schema, credentials, and any additional options. Every ingestion process starts by referencing this table to retrieve connection parameters.

* **`table_metadata`**  
   Holds table-specific configurations — table names, primary keys, partition columns, target paths, load frequency, optimization strategy (e.g., Z-Order), write modes, and caching options.

* **`column_metadata`**  
   Tracks detailed column-level information for each table — names, data types, nullable flags, primary key indicators, and target mappings.

* **`datamart_metadata`** and **`table_datamart_mapping`**  
   Enable logical grouping of tables into datamarts, supporting downstream analytics organization (To be implemented).

* **`ingestion_schedule`**  
   Defines how and when ingestion runs — cron expressions for scheduling, active flags, and last/next run timestamps.

* **`run_metadata`** and **`etl_run_logs`**  
   Provide full traceability of ingestion runs, including execution timestamps, modes, statuses, errors, and row counts.

### **3\. Modular Workflow Components**

The ingestion process is broken into reusable stages, each interacting with one or more metadata tables:

1. **Parameter Retrieval** → feeds into `connection_metadata` lookups.

2. **Connection Metadata Management** → creates or updates `connection_metadata`.

3. **Metadata Initialization** → ensures `table_metadata` and `column_metadata` exist.

4. **Table & Column Metadata Population** → populates schema definitions.

5. **Scheduling & Execution** → uses `ingestion_schedule` for automated runs or `run_metadata` for ad-hoc runs.

6. **Logging & Monitoring** → stores results in `etl_run_logs`.

### **4\. Built-in Automation Readiness**

* Cron expressions in `ingestion_schedule` allow precise timing for ingestion runs.

* Supports both **one-time** and **recurring** workflows without extra scripting.

### **5\. Conditional Logic for Efficiency**

* Prevents duplicate entries in `connection_metadata` and `table_metadata`.

* Skips metadata creation steps if already populated.

* Executes ingestion only if changes or schedules require it.

## **Step-by-Step Walkthrough**

### **Parameter Descriptions**

The utility uses the following parameters to configure connection settings, ingestion behavior, and target paths.  
 These parameters are typically set via **Databricks widgets** before execution.

#### Connection & Authentication Parameters

* **connection\_id** — Unique identifier for the connection configuration.

* **type** — Type of source database (e.g., `postgres`, `mysql`).

* **host** — Hostname or IP address of the source database server.

* **port** — Port number for database connection (integer).

* **database** — Name of the source database.

* **schema** — Schema name within the source database.

* **username** — Username for authentication.

* **password** — Password for authentication.

* **options** — Additional connection options in key-value format (optional).

---

#### Ingestion Control Parameters

* **table\_id** — Specific table ID to process (optional, for single-table operations).

* **cron\_expression** — Cron expression for scheduling ingestion jobs (optional).

* **schema\_name** — Schema to be ingested or processed.

* **target\_db** — Target database in Databricks, `abfss`, `s3`, or `dbfs` where data will be stored.

* **target\_prefix** — Prefix for target table names (optional).

* **target\_suffix** — Suffix for target table names (optional).

* **target\_path\_type** — Type of target path (optional, e.g., managed, external, `abfss`, `s3`, `dbfs`).

#### Target Storage Parameters

##### Azure Data Lake Gen2 (`abfss`)

* **target\_container** — Container name in your Azure storage account (**required**).

* **target\_account** — Azure storage account name (**required**).

* **tgt\_tbl\_name** — Target table or file name.

* **target\_db** *(optional)* — Used as a folder inside the container.

**Example path:**

`abfss://<target_container>@<target_account>.dfs.core.windows.net/<target_db>/<tgt_tbl_name>`

* 

##### Amazon S3 (`s3`)

* **target\_bucket** — S3 bucket name (**required**).

* **tgt\_tbl\_name** — Target table or file name.

* **target\_db** *(optional)* — Folder inside the bucket.

**Example path:**

`s3://<target_bucket>/<target_db>/<tgt_tbl_name>`

* 

##### Databricks File System (`dbfs`)

* **target\_mount** — DBFS mount point (**required**).

* **tgt\_tbl\_name** — Target table or file name.

* **target\_db** *(optional)* — Folder inside the mount.

##### **Example path:**   `/dbfs/mnt/<target_mount>/<target_db>/<tgt_tbl_name>`

##### Default (Local Mount or Other)

* Defaults to `/mnt/datalake` when no specific path type is set.

* **Example path:**

   php-template  
  CopyEdit  
  `/mnt/datalake/<target_db>/<tgt_tbl_name>`

### Notebooks:

#### 1\. Setup\_all\_with\_connection\_config:

Acts as the **starting notebook** for the entire process.

Reads all necessary parameters from widgets.

Sequentially calls:

1. **`metadata_tables_ddl`**

2. **`metadata_population_orchestration`**

3. **`ingestion_orchestration`**

#### 2\. metadata\_tables\_ddl

*(Purpose: One-time or as-needed run to ensure all metadata tables exist in the workspace)*

* Initializes all required metadata tables if they do not already exist.

#### 3\. Metadata\_population\_orchestration

* Retrieves parameters from widgets.

* Checks `connection_metadata`; if not found, inserts a new record.

* If `table_metadata` is missing, **calls `metadata_auto_populate`** to populate it.

#### 4\. metadata\_auto\_populate (invoked inside metadata\_population\_orchestration)

*(Purpose: Retrieve widget values, check connection metadata, populate table & column metadata if not available)*

* Inputs: widget values (connection type, host, port, schema, table list, etc.)

* Reads: `connection_metadata`, `table_metadata`

* Writes: updates/inserts into metadata tables

* Logic flow:

  1. Read widgets → connection params

  2. Check if connection already exists → load params or insert new row

  3. If no table metadata → trigger table & column metadata fetch

* Example widget set & resulting metadata rows

* Snippets: widget reading, metadata insertion, table listing

* Logging: note in `run_metadata` if population triggered

#### 5\. ingestion\_orchestration

* Reads parameters to determine execution type:

  * **Immediate ingestion** → Calls `metadata_driven_ingestion`.

  * **Scheduled ingestion** → Updates `ingestion_schedule` table and registers the job.

#### 6\. **metadata\_driven\_ingestion** *(invoked inside `ingestion_orchestration`)*

*(Purpose: Based on widget values, trigger immediate ingestion or schedule a recurring job)*

* Inputs: widgets (cron expression optional, ingestion mode, table list)

* Reads: `ingestion_schedule`, `table_metadata`, `connection_metadata`

* Writes: new schedule entry or triggers ingestion run, updates `etl_run_logs`

* Logic flow:

  1. Read widgets → ingestion mode decision (immediate vs scheduled)

  2. If scheduled → insert/update in `ingestion_schedule`

  3. If immediate → trigger ingestion executor logic

  4. Log run details into `run_metadata` & `etl_run_logs`

* Example: setting a cron for daily ingestion at midnight

* Example logs after a successful run**Conclusion**


This Databricks utility simplifies data ingestion by providing a robust, metadata-driven framework that eliminates repetitive coding and manual configurations. By centralizing connection and table details, automating schema discovery, and enabling both immediate and scheduled ingests, it empowers data teams to onboard new data sources faster, with greater consistency and reduced operational overhead. This framework fosters a truly agile data environment, allowing organizations to focus on deriving insights rather than managing the complexities of data pipelines.

## Resources

📂 **Source Code**  
Access the complete codebase for this Databricks ingestion utility:  
🔗 [GitHub Repository – Databricks Ingestion Utility](https://github.com/mustafasajid/databricks_enterprise_framework/tree/main)


## Future Enhancements

To extend the capabilities of the Databricks ingestion utility, several enhancements are planned:

1. **Expanded Data Source Support**

   * Integrate connectors for additional data sources such as NoSQL databases, REST/GraphQL APIs, and real-time streaming platforms (Kafka, Event Hubs, Kinesis).  
     

2. **Advanced Data Transformations** *(Transformation Layer)*

   * Introduce a metadata-driven transformation module for light ETL processes:

     * Data type casting

     * Standardizing column names

     * Basic aggregations and filters

     * Lightweight enrichment before loading to the serving layer

3. **Data Quality Checks**

   * Implement automated data quality validation based on predefined rules (e.g., null percentage checks, data type mismatches, threshold validations).

   * Trigger alerts via email, Teams, or Slack.

4. **Datamart Management** *(Serving Layer)*

   * Fully leverage `datamart_metadata` and `table_datamart_mapping` for logical grouping of datasets.

   * Apply access controls and organize curated datasets for BI tools and analytics consumers.

   

   

