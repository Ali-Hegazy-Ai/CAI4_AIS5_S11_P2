# ETL Pipeline

This page explains how the Azure Data Factory pipeline works end-to-end -- from reading raw source files to writing clean data into SQL Server.

---

## What Is a Pipeline?

In Azure Data Factory, a **Pipeline** is a sequence of activities that run in a defined order. Think of it like a recipe: each activity is one cooking step, and the pipeline runs them all in the right order.

Our pipeline is called `pl_customer_etl`.

---

## Pipeline Overview

```
[Source Files in data/raw/]
        |
        v
+-------+-------+
| Activity 1:   |  Copy CRM CSV from raw/ to staging
| Copy CRM      |
+-------+-------+
        |
        v
+-------+-------+
| Activity 2:   |  Copy Excel file from raw/ to staging
| Copy Excel    |
+-------+-------+
        |
        v
+-------+-------+
| Activity 3:   |  Merge, clean, deduplicate (see Data Flow section)
| Data Flow     |
+-------+-------+
        |
        +---------------------+
        v                     v
+-------+-------+   +---------+-------+
| Activity 4:   |   | Activity 5:     |
| Write CSV to  |   | Load rows into  |
| data/clean/   |   | SQL Server      |
+---------------+   +-----------------+
```

---

## ADF Assets Reference

All ADF configuration is stored as JSON in the repository. Never edit these JSON files by hand -- always make changes in ADF Studio and then export the updated JSON.

| Asset Type | Folder | Files |
|---|---|---|
| Pipelines | `adf/pipelines/` | `pl_customer_etl.json` |
| Data Flows | `adf/pipelines/` | `df_merge_transform.json` |
| Datasets | `adf/datasets/` | `ds_crm_source.json`, `ds_excel_source.json`, `ds_clean_output.json`, `ds_sql_customers.json` |
| Linked Services | `adf/linked_services/` | `ls_blob_storage.json`, `ls_sql_server.json` |

---

## Linked Services

A **Linked Service** is a connection definition -- it tells ADF how to connect to an external system.

| Name | Connects To | Used By |
|---|---|---|
| `ls_blob_storage` | Azure Blob Storage | Reading `data/raw/` files and writing `data/clean/` files |
| `ls_sql_server` | SQL Server database | Loading clean data into `dbo.Customers` |

---

## Datasets

A **Dataset** defines the structure and location of data that ADF reads or writes.

| Dataset Name | Type | Linked Service | Location / Table |
|---|---|---|---|
| `ds_crm_source` | DelimitedText (CSV) | `ls_blob_storage` | `data/raw/crm_customers_*.csv` |
| `ds_excel_source` | Excel | `ls_blob_storage` | `data/raw/excel_customers_*.xlsx` (sheet: `Customers`) |
| `ds_clean_output` | DelimitedText (CSV) | `ls_blob_storage` | `data/clean/customers_clean_YYYYMMDD.csv` |
| `ds_sql_customers` | SQL Server Table | `ls_sql_server` | `dbo.CustomerStaging` |

---

## Data Flow: df_merge_transform

The **Data Flow** is where all the actual data cleaning and merging happens. Data Flows run on a Spark cluster inside ADF -- you design them visually and ADF generates the Spark code.

### Data Flow Diagram

```
+------------------+          +------------------+
|  Source: CRM     |          |  Source: Excel   |
|  ds_crm_source   |          |  ds_excel_source |
+--------+---------+          +--------+---------+
         |                             |
         v                             v
+--------+---------+          +--------+---------+
|  Select          |          |  Select          |
|  Rename columns  |          |  Rename columns  |
|  to unified      |          |  to unified      |
|  schema          |          |  schema          |
+--------+---------+          +--------+---------+
         |                             |
         v                             v
+--------+---------+          +--------+---------+
|  Derived Column  |          |  Derived Column  |
|  Fix data types, |          |  Fix data types, |
|  formats,        |          |  formats,        |
|  add SourceSystem|          |  add SourceSystem|
+--------+---------+          +--------+---------+
         |                             |
         v                             v
+--------+---------+          +--------+---------+
|  Filter          |          |  Filter          |
|  Drop rows with  |          |  Drop rows with  |
|  null CustomerID |          |  null CustomerID |
|  or Name         |          |  or Name         |
+--------+---------+          +--------+---------+
         |                             |
         +-------------+---------------+
                       |
                       v
             +---------+---------+
             |  Union            |
             |  Combine both     |
             |  streams into one |
             +---------+---------+
                       |
                       v
             +---------+---------+
             |  Aggregate        |
             |  Deduplicate on   |
             |  CustomerID       |
             |  Keep most recent |
             +---------+---------+
                       |
              +--------+--------+
              v                 v
    +---------+------+  +-------+---------+
    |  Sink: CSV     |  |  Sink: SQL      |
    |  data/clean/   |  |  CustomerStaging|
    +----------------+  +-----------------+
```

### Transformation Rules

#### Step 1 -- Select (Column Renaming)

Both sources are unified to the same column names:

| CRM Column | Excel Column | Unified Column |
|---|---|---|
| `customer_id` | `CustomerID` | `CustomerID` |
| `full_name` | `Name` | `CustomerName` |
| `email` | `EmailAddress` | `Email` |
| `phone` | `PhoneNumber` | `Phone` |
| `signup_date` | `JoinDate` | `SignupDate` |
| `country` | `Country` | `Country` |
| `segment` | `CustomerSegment` | `Segment` |

#### Step 2 -- Derived Column (Business Rules)

| Column | Rule Applied |
|---|---|
| `CustomerID` | Cast to integer |
| `CustomerName` | `trim(initCap(CustomerName))` -- remove spaces and title-case |
| `Email` | `lower(trim(Email))` -- lowercase and trim |
| `Phone` | Regex normalisation to `+XX-XXX-XXXXXXX` format |
| `SignupDate` | Parse string to Date using `toDate(SignupDate, 'yyyy-MM-dd')` or `'dd/MM/yyyy'` |
| `Country` | Map ISO codes to full names using lookup table |
| `Segment` | `upper(Segment)` -- enforce uppercase |
| `SourceSystem` | Hardcoded as `'CRM'` or `'Excel'` depending on the stream |

#### Step 3 -- Filter (Drop Bad Rows)

Rows are dropped if:
- `CustomerID` is null or zero
- `CustomerName` is null or blank after trimming

Dropped rows are logged to `data/rejected/` for review.

#### Step 4 -- Union

The two cleaned streams (CRM and Excel) are appended into a single stream. At this point the combined stream may contain duplicate `CustomerID` values.

#### Step 5 -- Aggregate (Deduplicate)

Group by `CustomerID` and use `last()` / `first()` aggregation to pick one record per customer. The CRM record is preferred -- if both sources have the same `CustomerID`, the CRM values are used for key fields.

---

## Running the Pipeline

### Debug Run (Test with Sample Data)

1. Open ADF Studio and go to **Author > Pipelines**.
2. Open `pl_customer_etl`.
3. Click **Debug** in the toolbar.
4. ADF will sample the first 100 rows from each source and run through the entire pipeline.
5. Check the activity output at the bottom of the screen.

### Full Run (Production)

1. Click **Add Trigger > Trigger Now** to start a full run immediately.
2. Go to **Monitor > Pipeline Runs** to watch progress.
3. Click on the pipeline run to see individual activity results and row counts.

### Scheduled Run

See the [Project Architecture](Project-Architecture) page for instructions on setting up a scheduled or event-based trigger.

---

## After the Pipeline Runs

1. Check **Monitor > Pipeline Runs** -- status should be **Succeeded**.
2. Verify output in `data/clean/` -- a new CSV file should appear.
3. In SSMS, run: `SELECT COUNT(*) FROM dbo.Customers;` to verify rows were loaded.
4. Run the validation queries from `sql/scripts/05_validation_queries.sql`.
5. Update the progress table in `docs/project_flow.md`.

---

## Updating ADF Assets After Making Changes

Whenever you change a pipeline, dataset, or linked service in ADF Studio:

1. Click **Publish All** in ADF Studio to save changes to Azure.
2. Export the changed item as JSON (right-click > Export).
3. Replace the corresponding file in `adf/pipelines/`, `adf/datasets/`, or `adf/linked_services/`.
4. Commit and push the updated JSON to GitHub:

```bash
git add adf/
git commit -m "Update ADF pipeline: describe what you changed"
git push
```
