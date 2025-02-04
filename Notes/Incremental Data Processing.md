Incremental Data Processing

1\. Identify where Delta Lake provides ACID transactions  
Delta Lake provides ACID (Atomicity, Consistency, Isolation, Durability) transactions through its unique architecture and features.   
**Atomicity** ensures that all operations within a transaction either succeed completely or fail entirely.  
	df.write.format("delta").mode("overwrite").save("/path/to/delta\_table")  
Overwrite mode ensures the atomicity.  
**Consistency** ensures that the database transitions from one valid state to another.  
	spark.sql("SET spark.databricks.delta.schema.autoMerge.enabled \= true")  
This allows schema evolution while maintaining consistency.  
**Isolation** ensures that concurrent transactions do not interfere with each other. It implements isolation by Optimistic concurrency control and Serializable isolation.  
Multiple users writing to the same Delta table will not overwrite each other's data unless explicitly handled using Merge, Update and Delete.  
**Durability** ensures that once a transaction is committed, it remains persistent even in the event of a system failure. It maintains transaction logs as json files and immutable files which ensure durability.

2\. Identify the benefits of ACID transactions.

- Data integrity \> ensure that data remain accurate and consistent.  
- Fault tolerance \> protect against system failure.  
- Concurrent access \> allows multiple users or processes to work.  
- Schema enforcement \> prevents incompatible or corrupt data from being written to the table.  
- Time travel and auditability \> enables querying historical versions of the data and auditing changes over time.  
- Scalability \> enables scalable and reliable data processing in distributed environments.  
- Immutable data files \> ensures data durability and simplifies versioning.

3\. Compare and contrast data and metadata.  
Data : 

- Refers to the actual content or information stored in a system, such as records, files, or values in a database.  
- Represents the "what" – the raw facts or figures that are processed or analyzed.  
- Is the primary focus of storage, processing, and analysis in any system.  
- Used for analysis, reporting, decision-making, and deriving insights.  
- Subject to governance policies for privacy, security, and compliance.

Metadata : 

- Provides descriptive information about the data, such as its structure, format, source, or usage.  
- Represents the "how," "when," "where," and "why" – it helps organize, manage, and interpret the data.  
- Acts as secondary information that supports the understanding and management of the data.  
- Used for cataloging, searching, governance, and ensuring data quality.  
- Plays a critical role in enforcing governance by providing lineage, audit trails, and access control information.

4\. Compare and contrast managed and external tables.  
Managed Tables : 

- Fully controlled by the Databricks or Spark environment. The system manages both the metadata and the data files.  
- Data files are stored in a default location within the Databricks-managed storage (e.g., DBFS root).  
- When you drop a managed table, both the metadata and the underlying data files are deleted.  
- Ideal for scenarios where you want simplicity and full lifecycle management by the system, such as temporary tables, intermediate results in ETL pipelines, or when you don’t need to share data with external systems.  
- Often optimized for performance within the Databricks ecosystem, as the system has full control over data placement, caching, and optimization.  
- Primarily used within the Databricks ecosystem and may require additional steps to integrate with external tools.  
- Governance and security policies are applied directly through the Databricks environment, ensuring centralized control.

External Tables : 

- Only the metadata is managed by the system, while the data files are stored in an external location (e.g., cloud storage like AWS S3 or Azure Blob Storage) that you control.  
- Data files reside in a user-specified external location, often outside the Databricks-managed storage.  
- Dropping an external table only removes the metadata from the catalog; the actual data files remain intact in the external storage.  
- Best suited for use cases where data needs to be shared across multiple platforms, teams, or tools, or when you want to decouple data storage from compute.  
- Performance depends on the external storage system and how the data is organized. For example, using Delta Lake format with proper partitioning and ZOrdering can still achieve high performance.  
- Easily integrated with third-party tools, BI platforms, or other analytics systems.  
- Governance and security depend on the external storage system’s capabilities.

5\. Create a managed and external table.  
Managed table  
\# Create a managed table using PySpark  
data \= \[  
    (1, "Alice", 25),  
    (2, "Bob", 30),  
    (3, "Charlie", 35\)  
\]

columns \= \["id", "name", "age"\]

\# Create a DataFrame  
df \= spark.createDataFrame(data, columns)

\# Save the DataFrame as a managed table  
df.write.format("delta").saveAsTable("managed\_table\_example")

External table  
	\# Define the path to the external storage location  
external\_location \= "/mnt/external\_storage/external\_table\_example"  
\# Create a DataFrame  
data \= \[  
    (1, "Alice", 25),  
    (2, "Bob", 30),  
    (3, "Charlie", 35\)  
\]  
columns \= \["id", "name", "age"\]  
df \= spark.createDataFrame(data, columns)

\# Save the DataFrame as an external table  
df.write.format("delta").save(external\_location)

\# Register the external table in the metastore  
spark.sql(f"""  
CREATE TABLE external\_table\_example  
USING DELTA  
LOCATION '{external\_location}'  
""")

6\. Inspect the directory structure of Delta Lake files.  
Data Files : Stored in Parquet format, organized by partitioning if applicable.  
Transaction Log (\_delta\_log) : Records all changes to the table, enabling ACID compliance, time travel, and auditability.  
Partitioning : Improves query performance by organizing data into subdirectories based on partition columns.  
Time Travel : Enabled by the transaction log, allowing you to query historical versions of the table.  
File Management : Features like OPTIMIZE and VACUUM ensure efficient storage and performance.

7\. Query a specific version of a table.  
In Delta Lake, you can query a specific version of a table using time travel . Time travel allows you to access historical versions of the data by specifying either a version number or a timestamp . This is particularly useful for auditing, debugging, or recovering previous states of the table.  
SELECT \* FROM table\_name VERSION AS OF version\_number  
Example \- SELECT \* FROM DELTA.\`EMPLOYEE\` VERSION AS OF 3  
   \-  SELECT \* FROM DELTA.\`EMPLOYEE\`@v3

8\. Query a specific timestamp of a table.  
SELECT \* FROM table\_name TIMESTAMP AS OF 'yyyy-MM-ddTHH:mm:ss.SSSZ'  
Example \- SELECT \* FROM DELTA.\`EMPLOYEE\` TIMESTAMP AS OF '2025-01-01T06:30:00.000Z'

9\. Identify why Zordering is beneficial to Delta Lake tables.  
ZOrdering is a powerful optimization technique in Delta Lake that improves query performance by co-locating related data in the same files. This reduces the amount of data scanned during queries, leading to faster and more efficient operations.

- Improve query performance.  
- Enhance data skipping.  
- Optimized storage layout.  
- Reduced I/O cost.  
- Compatibility with partitioning.  
- Ideal for large tables.

10\. Identify how vacuum commits deletes.  
The VACUUM command in Delta Lake is used to clean up old files that are no longer needed, such as files left over from updates, deletes, or overwrites. These files are marked as "tombstoned" during operations like DELETE or UPDATE, but they are not physically removed until the VACUUM command is executed.  
Example \- 	VACUUM my\_table;  
		VACUUM my\_table RETAIN 168 HOURS; \-- Retain files for 7 days (168 hours)

11\. Identify the kind of files Optimize compacts.   
The OPTIMIZE command in Delta Lake is used to improve the performance of a table by compacting small files into larger ones. This process, also known as file compaction , reduces the number of files in the table, which improves query performance and reduces metadata overhead.  
Why Are Small Files a Problem?  
Increased I/O Overhead : Queries need to read many small files, which increases the number of I/O operations and slows down performance.

Metadata Overhead : Delta Lake maintains metadata for each file, so having too many small files increases the size of the transaction log (\_delta\_log) and slows down operations like listing files.

Inefficient Data Skipping : Delta Lake uses statistics (e.g., min/max values) to skip irrelevant files during queries. With many small files, this optimization becomes less effective.

12\. Identify CTAS as a solution.  
CTAS (Create Table As Select) is a powerful SQL construct used to create a new table and populate it with the results of a SELECT query. It is widely used in data engineering workflows for tasks like data transformation , ETL processes , and table optimization.  
Example \-CREATE TABLE new\_table\_name  
     USING delta  
 	     AS SELECT column1, column2, ...  
  	     FROM source\_table  
     WHERE conditions;  
The new table inherits the schema and data types from the SELECT query.  
Use cases \- 	Data transformation  
		Schema evolution  
		Partitioning and Zordering  
		Data archiving  
		Cloning tables

13\. Create a generated column.  
In Delta Lake, generated columns are a powerful feature that allows you to define columns whose values are automatically computed based on expressions involving other columns in the table. These columns are useful for deriving values (e.g., computed fields) without requiring manual updates or additional transformations during queries.  
Characteristics :

- The value of a generated column is computed automatically when data is inserted or updated.  
- Generated columns cannot be directly modified by users; their values are always determined by the defined expression.  
- They are stored physically in the table, which improves query performance compared to computing the value on-the-fly.

Basic syntax (SQL)  
CREATE TABLE my\_table (  
    id INT,  
    first\_name STRING,  
    last\_name STRING,  
    full\_name STRING GENERATED ALWAYS AS (concat(first\_name, ' ', last\_name))  
) USING DELTA;

14\. Compare and contrast CREATE OR REPLACE TABLE and INSERT OVERWRITE  
CREATE OR REPLACE TABLE  
Purpose : Replaces the entire table, including its schema and data.  
Use Case : Useful when you want to redefine the table structure (schema) and populate it with new data.  
Behavior :  
Drops the existing table (if it exists) and creates a new table with the specified schema.  
Overwrites all metadata and data associated with the table.  
Data Replacement : Completely replaces the data in the table.  
All existing data is deleted, and the table is populated with new data.  
Typically slower because it involves dropping and recreating the table.

INSERT OVERWRITE  
Purpose : Replaces only the data in an existing table while preserving the table's schema.  
Use Case : Useful when you want to overwrite specific partitions or the entire dataset without altering the table's structure.  
Behavior :  
Writes new data into the table, replacing existing rows or partitions.  
Retains the table's schema and metadata.  
Data Replacement : Overwrites specific data within the table.  
If the table is partitioned, INSERT OVERWRITE can target specific partitions.  
If the table is unpartitioned, it overwrites the entire dataset.  
Faster for updating specific partitions or datasets, as it avoids recreating the entire table.

15\. Identify a scenario in which MERGE should be used.

1. Change Data Capture (CDC)  
2. Data Deduplication  
3. Slowly Changing Dimensions (SCD)  
4. Real-Time Data Ingestion

16\. Describe the benefits of the MERGE command.

- Ensures that all operations (inserts, updates, deletes) are performed as a single transaction.  
- Combines multiple operations (inserts, updates, deletes) into a single command.  
- Ensures that repeated executions of the same MERGE operation do not create duplicates.  
- Allows you to define custom logic for handling matches and mismatches between the source and target tables.  
- ensures that all changes made by the MERGE command adhere to ACID (Atomicity, Consistency, Isolation, Durability) principles.  
- Optimized for large-scale datasets and distributed systems like Delta Lake.  
- Reduces the complexity of ETL workflows by consolidating multiple operations into one step.  
- Facilitates the application of changes (inserts, updates, deletes) from a source system to a target Delta table.

17\. Identify why a COPY INTO statement is not duplicating data in the target table.  
The COPY INTO statement in Delta Lake is designed to efficiently load data from cloud storage (e.g., AWS S3, Azure Blob Storage, or Google Cloud Storage) into a Delta table. One of its key features is that it ensures idempotency , meaning it avoids duplicating data in the target table even if the same source files are processed multiple times.  
COPY INTO is particularly useful for incremental data loading scenarios, where new files are periodically added to the source location.  
Example \- 	COPY INTO target\_table  
FROM 's3://source-bucket/data/'  
FILEFORMAT \= PARQUET;

18\. Identify a scenario in which COPY INTO should be used. 

- Incremental Data Loading  
- Idempotent Data Ingestion(without duplicate rows)  
- Bulk Data Migration  
- Schema Enforcement and Evolution  
- Streaming Data Archival  
- Data Replication Across Regions  
- Backfilling Historical Data  
- Handling Large File Sets  
- Multi-Format Data Ingestion  
- Fault-Tolerant Pipelines


19\. Compare and contrast triggered and continuous pipelines in terms of cost and latency.  
Triggered pipeline : 

- A triggered pipeline runs on-demand or at scheduled intervals (e.g., hourly, daily).  
- It processes all available data since the last run and then stops.  
- Triggered pipelines are cost-effective by reducing compute resource usage.  
- Ideal for batch processing workloads where real-time updates are not required.

Use Cases

- Batch ETL workflows.  
- Periodic data aggregation or reporting.  
- Scenarios where real-time processing is not critical.

Continuous pipeline : 

- A continuous pipeline runs non-stop, processing data as soon as it arrives in the source location.  
- It is designed for real-time or near-real-time data ingestion and processing.  
- Continuous pipelines require compute resources to remain active at all times, leading to higher costs.  
- Suitable for scenarios where the benefits of real-time processing justify the additional expense.  
- Ideal for use cases requiring immediate insights or actions based on incoming data.

Use Cases

- Real-time analytics (e.g., monitoring dashboards).  
- Streaming data ingestion (e.g., IoT sensors, clickstream data).  
- Event-driven architectures where timely processing is critical.

20\. Identify the default behavior of a constraint violation  
In Delta Live Tables (DLT) , constraints are used to enforce data quality rules by validating the data as it flows through the pipeline. When a constraint violation occurs (e.g., a row fails to meet the specified condition), the default behavior is to drop the violating rows from the dataset.  
Why It Matters :

- This ensures that invalid or low-quality data does not pollute the target table.  
- The pipeline continues running without interruption, even if some rows fail the constraint.

Example \- 	@dlt.table  
@dlt.expect("valid\_age", "age \> 0")  
def users():  
    			return spark.read.format("delta").load("/path/to/users")

21\. Identify the impact of ON VIOLATION DROP ROW and ON VIOLATION FAIL UPDATEfor a constraint violation  
constraints are used to enforce data quality rules by validating rows as they flow through the pipeline. When a constraint violation occurs, the behavior of the pipeline depends on how the constraint is configured. Specifically, the options ON VIOLATION DROP ROW and ON VIOLATION FAIL UPDATE define how the pipeline handles violations.  
ON VIOLATION DROP ROW : 

- Rows that violate the constraint are dropped from the dataset.  
- Only rows that satisfy the constraint are written to the target Delta table.  
- The pipeline continues running without interruption.

Use Case

- Suitable for pipelines where some bad data is acceptable, and you want to ensure that the pipeline remains operational.  
- Example: Ingesting raw data from an external source where occasional invalid rows are   
  expected.

ON VIOLATION FAIL UPDATE :

- If any row violates the constraint, the entire pipeline fails .  
- No data is written to the target table, and the pipeline stops processing further data.

Use Case

- Suitable for pipelines where data quality is critical, and any violation must be addressed immediately.  
- Example: Processing financial or compliance-related data where even a single invalid row   
  is unacceptable.

22\. Explain change data capture and the behavior of APPLY CHANGES INTO  
Change Data Capture (CDC) is a technique used to track and capture changes made to data in a source system, such as inserts, updates, and deletes. The goal of CDC is to efficiently propagate these changes to downstream systems, such as data warehouses, data lakes, or analytical databases, without reprocessing the entire dataset. This ensures that downstream systems remain synchronized with the source system in near real-time.

Key Benefits of CDC

- Efficiency : Only changed data is processed, reducing the volume of data transferred and improving performance.  
- Real-Time Synchronization : Enables downstream systems to stay up-to-date with minimal latency.  
- Auditability : Captures a complete history of changes, which can be useful for auditing, debugging, or replaying historical data.  
- Minimized Impact on Source Systems : Reduces the load on source systems compared to full data reloads.

APPLY CHANGES INTO  
In Delta Lake, the APPLY CHANGES INTO command is a powerful tool for implementing Change Data Capture (CDC) . It allows you to apply changes from a source table (or stream) containing change events into a target Delta table. The command processes the changes incrementally, ensuring that the target table reflects the latest state of the source data.

How APPLY CHANGES INTO Works  
The APPLY CHANGES INTO command processes change events (inserts, updates, and deletes) from a source table or stream and applies them to a target Delta table.

23\. Query the events log to get metrics, perform audit loggin, examine lineage.  
In Delta Lake , the event log (also known as the Delta transaction log or \_delta\_log) is a critical component that tracks all changes made to a Delta table. This log contains metadata about every operation performed on the table, such as inserts, updates, deletes, schema changes, and more. Querying the event log allows you to extract valuable information for metrics analysis , audit logging , and data lineage examination.  
Reading the event log :   
	from pyspark.sql.functions import col  
\# Path to the Delta table  
delta\_table\_path \= "/path/to/delta\_table"  
\# Read the Delta log as a DataFrame  
delta\_log\_df \= spark.read.format("json").load(f"{delta\_table\_path}/\_delta\_log/\*.json")  
\# Display the contents of the log  
delta\_log\_df.show(truncate=False)

Extracting metrics : The event log contains metadata about each operation, including metrics  
	\# Filter for "add" actions to get metrics about new files  
add\_actions \= delta\_log\_df.filter(col("add").isNotNull())

\# Extract relevant metrics  
metrics\_df \= add\_actions.select(  
    		col("add.path").alias("file\_path"),  
    		col("add.size").alias("file\_size"),  
   		col("add.dataChange").alias("is\_data\_change"),  
    		col("add.stats.numRecords").alias("num\_records")  
)  
metrics\_df.show()

Audit logging : it involves tracking who performed what operation on the table and when.  
	\# Extract audit information from commitInfo  
audit\_log\_df \= delta\_log\_df.select(  
    		col("commitInfo.operation").alias("operation"),  
    		col("commitInfo.userName").alias("user"),  
    		col("commitInfo.timestamp").alias("timestamp"),  
    		col("commitInfo.notebook").alias("notebook"),  
    		col("commitInfo.job").alias("job")  
)  
audit\_log\_df.show(truncate=False)

Examining data lineage : Data lineage refers to the flow of data through transformations and operations.  
\# Extract lineage information by ordering operations by timestamp  
lineage\_df \= delta\_log\_df.select(  
    col("commitInfo.operation").alias("operation"),  
    col("commitInfo.timestamp").alias("timestamp"),  
    col("commitInfo.operationMetrics").alias("metrics")  
).orderBy("timestamp")  
lineage\_df.show(truncate=False)

24\. Troubleshoot DLT syntax: Identify which notebook in a DLT pipeline produced an error,  
identify the need for LIVE in create statement, identify the need for STREAM in form clause.  
In a DLT pipeline, multiple notebooks or SQL scripts may define tables and transformations. When an error occurs, it can be challenging to identify which notebook caused the issue.

