# **Data Contract \- Metadata Tables**

This document defines the technical and business-friendly data contracts for all metadata tables used in the Databricks ingestion and metadata management framework.

## **connection\_metadata**

| Column Name | Data Type | Technical Description | Business-Friendly Description |
| :---- | :---- | :---- | :---- |
| connection\_id | STRING | Unique identifier for the connection. | Identifies the connection configuration. |
| type | STRING | Type of the source system (e.g., postgres, mysql). | Specifies the type of database. |
| host | STRING | Hostname or IP of the database server. | Where the database is hosted. |
| port | INT | Port number to connect to the database. | Port used for the connection. |
| database | STRING | Database name in the source system. | Name of the database to connect. |
| schema | STRING | Schema name inside the database. | Logical grouping of tables. |
| username | STRING | Username for authentication. | Login user for the source database. |
| password | STRING | Password for authentication. | Password for the source database. |
| options | STRING | Additional connection options. | Extra parameters for connection. |

## **table\_metadata**

| Column Name | Data Type | Technical Description | Business-Friendly Description |
| :---- | :---- | :---- | :---- |
| table\_id | STRING | Unique table identifier. | Identifies the table in the framework. |
| connection\_id | STRING | Foreign key to connection\_metadata. | Which connection this table belongs to. |
| table\_name | STRING | Name of the source table. | Original table name. |
| table\_call\_name | STRING | Alias or logical call name. | Used internally to reference table. |
| target\_table\_name | STRING | Target table name in Databricks. | Where the ingested table will be stored. |
| table\_type | STRING | Type of table (e.g., BASE, VIEW). | Source table classification. |
| primary\_key\_columns | STRING | Comma-separated list of primary keys. | Columns used to uniquely identify records. |
| watermark\_column | STRING | Column for incremental loads. | Tracks last loaded record for delta loads. |
| partition\_column | STRING | Column used for partitioning target data. | Improves performance by partitioning. |
| target\_path | STRING | Path to store table data. | Location in storage for the table. |
| load\_frequency | STRING | How often to load table. | Load schedule for the table. |
| active\_flag | STRING | Flag indicating if table is active. | Controls if table is processed. |
| comments | STRING | Additional notes. | Business notes or description. |
| optimize\_zorder\_by | STRING | Columns for Z-Ordering. | Columns for optimizing queries. |
| repartition\_columns | STRING | Columns to repartition data. | Improves write performance. |
| num\_output\_files | STRING | Number of output files expected. | Write tuning parameter. |
| write\_mode | STRING | Write mode (append/overwrite). | Controls overwrite or append writes. |
| cache\_intermediate | STRING | Cache intermediate results flag. | Improves performance for transformations. |
| target\_db | STRING | Target database name. | Database in Databricks or storage target. |
| onboarded\_flag | STRING | Flag for onboarding completion. | Indicates table onboarding status. |

## **column\_metadata**

| Column Name | Data Type | Technical Description | Business-Friendly Description |
| :---- | :---- | :---- | :---- |
| table\_id | STRING | FK to table\_metadata. | Which table the column belongs to. |
| column\_name | STRING | Original column name. | Source system column name. |
| data\_type | STRING | Source data type. | Data type in source system. |
| target\_type | STRING | Target data type. | Transformed data type. |
| target\_column\_name | STRING | Target column name. | Renamed column in target. |
| nullable | BOOLEAN | Whether column is nullable. | Indicates if null values are allowed. |
| is\_primary\_key | STRING | Primary key indicator. | Y/N if column is part of PK. |

## **datamart\_metadata**

| Column Name | Data Type | Technical Description | Business-Friendly Description |
| :---- | :---- | :---- | :---- |
| datamart\_id | STRING | Unique datamart identifier. | ID for the datamart. |
| datamart\_name | STRING | Name of the datamart. | Business-friendly datamart name. |
| description | STRING | Description of datamart. | Business purpose of the datamart. |

## **table\_datamart\_mapping**

| Column Name | Data Type | Technical Description | Business-Friendly Description |
| :---- | :---- | :---- | :---- |
| table\_id | STRING | FK to table\_metadata. | Links a table to a datamart. |
| datamart\_id | STRING | FK to datamart\_metadata. | Links datamart to the table. |

## **byod\_table\_catalog**

| Column Name | Data Type | Technical Description | Business-Friendly Description |
| :---- | :---- | :---- | :---- |
| catalog\_id | STRING | Unique catalog entry ID. | Identifier for BYOD catalog entry. |
| connection\_id | STRING | FK to connection\_metadata. | Connection source for catalog table. |
| schema\_name | STRING | Schema in source DB. | Schema containing the table. |
| table\_name | STRING | Name of table in source DB. | Source table name. |
| onboarded\_flag | STRING | Onboarded status. | Indicates if table is ready for use. |

## **ingestion\_schedule**

| Column Name | Data Type | Technical Description | Business-Friendly Description |
| :---- | :---- | :---- | :---- |
| schedule\_id | STRING | Unique schedule ID. | Identifies ingestion schedule. |
| connection\_id | STRING | FK to connection\_metadata. | Connection linked to schedule. |
| table\_id | STRING | FK to table\_metadata. | Table linked to schedule. |
| schedule\_type | STRING | Schedule level. | Whether schedule is for table or connection. |
| cron\_expression | STRING | Cron string for scheduling. | Defines ingestion timing. |
| active\_flag | STRING | Active status of schedule. | Enables/disables the schedule. |
| last\_run | TIMESTAMP | Timestamp of last run. | When ingestion last occurred. |
| next\_run | TIMESTAMP | Timestamp of next run. | When ingestion will occur next. |
| comments | STRING | Additional notes. | Extra details for schedule. |

## **run\_metadata**

| Column Name | Data Type | Technical Description | Business-Friendly Description |
| :---- | :---- | :---- | :---- |
| run\_id | STRING | Unique run ID. | Identifies ingestion run. |
| connection\_id | STRING | FK to connection\_metadata. | Connection linked to the run. |
| dateandtime | TIMESTAMP | Execution timestamp. | When ingestion was triggered. |
| user | STRING | User initiating run. | Person/system triggering ingestion. |
| ingest\_mode | STRING | Ingestion mode. | Type of ingestion performed. |

## **etl\_run\_logs**

| Column Name | Data Type | Technical Description | Business-Friendly Description |
| :---- | :---- | :---- | :---- |
| transaction\_id | STRING | Unique transaction ID. | ID for specific ingestion transaction. |
| run\_id | STRING | FK to run\_metadata. | Links to ingestion run. |
| connection\_id | STRING | FK to connection\_metadata. | Connection used in run. |
| table\_id | STRING | FK to table\_metadata. | Table processed in run. |
| table\_name | STRING | Table name processed. | Name of table in run. |
| dateandtime | STRING | Run timestamp. | When ingestion happened. |
| mode | STRING | Ingestion mode. | Full/Incremental mode. |
| status | STRING | Run status. | Success/Failure status. |
| error | STRING | Error message. | Failure details if any. |
| target\_table | STRING | Target table name. | Name in target system. |
| number\_of\_rows | BIGINT | Row count processed. | Number of rows ingested. |
| other\_comments | STRING | Additional notes. | Extra details on the run. |

 

