# Glossary

This page defines every technical term used in this project in plain English. If you are new to data engineering, start here.

---

## ETL Terms

### ETL
**Extract, Transform, Load** -- the three-stage process at the heart of this project.
- **Extract**: Read data from source systems (CRM CSV, Excel file)
- **Transform**: Clean, standardize, merge, and deduplicate the data
- **Load**: Write the clean data to the target database (SQL Server)

### ELT
A variant of ETL where data is loaded first, then transformed inside the database. This project uses the traditional ETL pattern where transformation happens before loading.

### Data Pipeline
A sequence of automated steps that moves data from one place to another, applying transformations along the way. In this project, the pipeline is `pl_customer_etl` in Azure Data Factory.

### Orchestration
The process of managing and scheduling when pipeline activities run and in what order. Azure Data Factory is the orchestration tool in this project.

### Medallion Architecture
A data organisation pattern with three layers:
- **Bronze**: Raw, unprocessed data (`data/raw/`)
- **Silver**: Cleaned and transformed data (ADF Data Flow output)
- **Gold**: Ready-for-analytics data (`data/clean/` and SQL Server)

### Data Lineage
The record of where each piece of data came from and what transformations it went through. Tracked here via the `SourceSystem` column and the `docs/project_flow.md` file.

### Idempotent
A script or process is idempotent if running it multiple times produces the same result as running it once. In SQL, this means using `IF NOT EXISTS` or `CREATE OR ALTER` so scripts do not fail if run twice.

---

## Azure Data Factory Terms

### Azure Data Factory (ADF)
Microsoft's cloud-based data integration service. ADF lets you build pipelines that move and transform data across many sources and destinations without writing server-side code.

### Pipeline
The top-level container for a workflow in ADF. A pipeline contains one or more activities that run in sequence or in parallel. This project's main pipeline is `pl_customer_etl`.

### Activity
A single step inside a pipeline. Examples:
- **Copy Activity**: Copies data from a source to a destination
- **Data Flow Activity**: Runs a Data Flow transformation
- **Execute Pipeline Activity**: Calls another pipeline

### Data Flow
A visual transformation engine inside ADF that runs on Apache Spark. You design transformations using drag-and-drop steps, and ADF generates the Spark code. This project's main Data Flow is `df_merge_transform`.

### Linked Service
A connection definition in ADF -- it stores the credentials and endpoint details for connecting to an external system (e.g. Azure Blob Storage, SQL Server). Think of it as a saved connection string.

### Dataset
A reference to a specific data location and its schema in ADF -- for example, "the CSV file at `data/raw/crm_customers_*.csv` with these column names and types". Datasets reference a Linked Service for the connection.

### Trigger
Tells ADF when to run a pipeline automatically. Types:
- **Schedule Trigger**: Runs on a time schedule (e.g. every Monday at 06:00)
- **Event Trigger**: Runs when a specific event occurs (e.g. a file arrives in Blob Storage)
- **Manual Trigger (Trigger Now)**: Runs immediately on demand

### Debug Mode
A test mode in ADF where the pipeline runs using only a sample of data (first 100 rows by default). Use Debug mode to test changes without processing full files.

### Publish
The action in ADF Studio that saves your pipeline/dataset/linked service changes to Azure. Changes are not active until you click **Publish All**.

---

## SQL Terms

### Schema (SQL)
In SQL Server, a schema is a namespace for database objects. This project uses the default `dbo` schema (e.g. `dbo.Customers`, `dbo.vw_CustomerSummary`).

### Primary Key
A column (or set of columns) whose value uniquely identifies each row in a table. In this project, `CustomerID` is the primary key of `dbo.Customers`. Primary keys cannot be NULL or duplicate.

### MERGE (Upsert)
A SQL operation that inserts a row if it does not exist or updates it if it does -- based on a matching key. Used in `dbo.usp_UpsertCustomers` to load data without duplicating rows.

### Staging Table
A temporary table used as a buffer during data loading. The pipeline loads transformed data into `dbo.CustomerStaging` first, then merges it into `dbo.Customers`, then truncates the staging table.

### View
A saved SQL query that you can query like a table. Views do not store data -- they re-run their underlying query each time. `dbo.vw_CustomerSummary` is an example.

### Stored Procedure
A saved, named SQL script that you can call by name. In this project, `dbo.usp_UpsertCustomers` is a stored procedure that handles the MERGE logic.

### TRUNCATE
A SQL command that deletes all rows from a table quickly. Used to clear `dbo.CustomerStaging` after each pipeline run. Faster than `DELETE` because it does not log individual row deletions.

---

## Data Quality Terms

### Deduplication
Removing duplicate records. In this project, if both the CRM and Excel sources have a record for the same `CustomerID`, the pipeline keeps only one -- preferring the CRM version.

### Null
A missing or unknown value in a database. Different from an empty string or zero. This project rejects rows where `CustomerID` is NULL because a customer without an ID cannot be reliably tracked.

### Data Type Casting
Converting a value from one data type to another. For example, converting the string `"2023-05-10"` to a proper SQL `DATE` value, or converting `"1001"` (string) to `1001` (integer).

### Validation Query
A SQL query written specifically to check whether data meets quality standards (e.g. no nulls, no duplicates, values in expected ranges). This project's validation queries are in `sql/scripts/05_validation_queries.sql`.

### Rejected Record
A row that fails a validation check and is excluded from the clean output. Rejected records should be captured and logged (e.g. in `data/rejected/`) so they can be investigated.

---

## Git / GitHub Terms

### Branch
An independent line of development in Git. You create a branch to work on a feature or fix without affecting `main`. When the work is done, you merge the branch back.

### Pull Request (PR)
A request to merge your branch into `main`. Pull requests are reviewed by teammates before merging to catch bugs and ensure quality.

### Commit
A saved snapshot of your changes in Git. Each commit has a message describing what changed.

### Merge
Combining changes from one branch into another. In this project, pull requests are merged into `main` after review approval.

### Clone
Downloading a copy of the repository to your local machine. You only do this once per machine: `git clone <url>`.

### Push
Uploading your local commits to GitHub so others can see them and you can open a pull request.

### Issue
A GitHub feature for tracking tasks, bugs, and feature requests. Every piece of work in this project starts as an issue.
