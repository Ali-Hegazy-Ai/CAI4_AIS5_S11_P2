# Project Architecture

This page explains every layer of the Customer Data ETL system in detail, so you can understand how data moves from raw source files to a clean, query-ready SQL table.

---

## Architecture Overview

The system follows a classic **Medallion Architecture** pattern:

```
[Raw Sources]  -->  [Bronze: raw/]  -->  [Silver: ADF Transform]  -->  [Gold: clean/ + SQL]
```

| Layer | Name | Description |
|---|---|---|
| Raw | Bronze | Unmodified source files land in `data/raw/` |
| Transform | Silver | ADF Data Flow cleans, merges, and standardizes |
| Load | Gold | Clean CSV in `data/clean/` and rows in SQL Server |

---

## Full Architecture Diagram

```
+=====================================================================+
|                         EXTRACT LAYER                               |
|                                                                     |
|  +------------------------+    +---------------------------+        |
|  |   CRM System           |    |   Excel Spreadsheet       |        |
|  |   (exported CSV)       |    |   (.xlsx file)            |        |
|  |                        |    |                           |        |
|  |  - customer_id         |    |  - CustomerID             |        |
|  |  - full_name           |    |  - Name                   |        |
|  |  - email               |    |  - EmailAddress           |        |
|  |  - phone               |    |  - PhoneNumber            |        |
|  |  - signup_date         |    |  - JoinDate               |        |
|  |  - country             |    |  - Country                |        |
|  |  - segment             |    |  - CustomerSegment        |        |
|  +----------+-------------+    +----------+----------------+        |
|             |                             |                         |
|             v                             v                         |
|        data/raw/crm_*.csv          data/raw/excel_*.xlsx           |
+=====================================================================+
                          |
                          v
+=====================================================================+
|                       TRANSFORM LAYER                               |
|                   (Azure Data Factory)                              |
|                                                                     |
|   Pipeline: pl_customer_etl                                         |
|   +---------------------------------------------------------------+ |
|   |                                                               | |
|   |  Activity 1: Copy CRM data to staging area                   | |
|   |  Activity 2: Copy Excel data to staging area                 | |
|   |  Activity 3: Data Flow -- df_merge_transform                 | |
|   |                                                               | |
|   |  DATA FLOW STEPS:                                             | |
|   |  +-------------------+    +-------------------+              | |
|   |  |  Source: CRM      |    |  Source: Excel    |              | |
|   |  +--------+----------+    +--------+----------+              | |
|   |           |                        |                         | |
|   |           v                        v                         | |
|   |  +--------+----------+    +--------+----------+              | |
|   |  |  Select           |    |  Select           |              | |
|   |  |  (rename cols)    |    |  (rename cols)    |              | |
|   |  +--------+----------+    +--------+----------+              | |
|   |           |                        |                         | |
|   |           v                        v                         | |
|   |  +--------+----------+    +--------+----------+              | |
|   |  |  Derived Column   |    |  Derived Column   |              | |
|   |  |  (fix types)      |    |  (fix types)      |              | |
|   |  +--------+----------+    +--------+----------+              | |
|   |           |                        |                         | |
|   |           v                        v                         | |
|   |  +--------+----------+    +--------+----------+              | |
|   |  |  Filter           |    |  Filter           |              | |
|   |  |  (drop bad rows)  |    |  (drop bad rows)  |              | |
|   |  +--------+----------+    +--------+----------+              | |
|   |           |                        |                         | |
|   |           +----------+-------------+                         | |
|   |                      v                                        | |
|   |             +--------+----------+                             | |
|   |             |  Union (merge)    |                             | |
|   |             +--------+----------+                             | |
|   |                      v                                        | |
|   |             +--------+----------+                             | |
|   |             |  Aggregate        |                             | |
|   |             |  (deduplicate on  |                             | |
|   |             |   CustomerID)     |                             | |
|   |             +--------+----------+                             | |
|   |                      |                                        | |
|   |          +-----------+-----------+                            | |
|   |          v                       v                            | |
|   |  +-------+-------+    +----------+--------+                  | |
|   |  |  Sink: CSV    |    |  Sink: SQL Server |                  | |
|   |  +---------------+    +-------------------+                  | |
|   |                                                               | |
|   +---------------------------------------------------------------+ |
+=====================================================================+
                          |
          +---------------+---------------+
          v                               v
+=====================================================================+
|                         LOAD LAYER                                  |
|                                                                     |
|  +---------------------------+    +----------------------------+    |
|  |  data/clean/              |    |  SQL Server                |    |
|  |  customers_clean.csv      |    |  CustomerDW database       |    |
|  |                           |    |                            |    |
|  |  Ready for:               |    |  Tables:                   |    |
|  |  - Power BI import        |    |  - dbo.Customers           |    |
|  |  - Excel analysis         |    |  - dbo.CustomerStaging     |    |
|  |  - Further processing     |    |  - dbo.ETLRunLog           |    |
|  +---------------------------+    |                            |    |
|                                   |  Views:                    |    |
|                                   |  - dbo.vw_CustomerSummary  |    |
|                                   +----------------------------+    |
+=====================================================================+
```

---

## Data Sources

Two source systems feed this pipeline:

### Source 1 -- CRM Export (CSV)

The CRM system generates a periodic CSV dump of all customer records. This file is manually placed in `data/raw/` and named following the convention `crm_customers_YYYYMMDD.csv`.

**Characteristics:**
- UTF-8 encoded comma-delimited text
- Columns use snake_case naming (`customer_id`, `full_name`)
- Date format: `YYYY-MM-DD`
- Contains older records with missing `signup_date` values
- Phone numbers in various local formats (no country code)

### Source 2 -- Excel Spreadsheet

The sales team maintains customer records in an Excel file updated weekly. This file is placed in `data/raw/` and named `excel_customers_YYYYMMDD.xlsx`.

**Characteristics:**
- Sheet name: `Customers` (always first sheet)
- Columns use PascalCase naming (`CustomerID`, `Name`)
- Date format: `DD/MM/YYYY` or Excel serial numbers
- Country values use 3-letter ISO codes instead of full names
- Phone numbers include country code prefix (`+20-...`)

Both sources share the `CustomerID` / `customer_id` field as the primary key for deduplication.

---

## Extraction Layer

The extraction layer is handled by **two ADF Copy Activities** inside `pl_customer_etl`:

| Activity | Source | Destination | Notes |
|---|---|---|---|
| `CopyCRMData` | `ds_crm_source` (CSV) | Staging blob path | Reads file from `data/raw/` |
| `CopyExcelData` | `ds_excel_source` (Excel) | Staging blob path | Reads first sheet |

No transformations happen here. Data is copied as-is into a temporary staging location so the Data Flow can read both in parallel.

---

## Transformation Layer

The transformation is performed by an **ADF Data Flow** named `df_merge_transform`. A Data Flow is a visual, code-free transformation engine inside Azure Data Factory.

### Step-by-Step Transformations

| Step | Transformation Type | What It Does |
|---|---|---|
| 1 | **Select** | Renames columns from both sources to a unified schema (`CustomerID`, `CustomerName`, `Email`, etc.) |
| 2 | **Derived Column** | Applies business rules: trim whitespace, lowercase emails, parse dates, normalize phone format, uppercase `Segment` values |
| 3 | **Filter** | Drops rows where `CustomerID IS NULL` or `CustomerName` is blank |
| 4 | **Union** | Appends the two cleaned streams into one combined dataset |
| 5 | **Aggregate** | Groups by `CustomerID` and keeps the most recent record, resolving duplicates between the two sources |

### Business Rules Applied

- Emails are converted to lowercase
- `CustomerName` has leading/trailing spaces removed and is title-cased
- Phone numbers are normalized to `+XX-XXX-XXXXXXX` format where possible
- `SignupDate` is parsed and stored as ISO `YYYY-MM-DD`
- `Country` is standardized to full English country name (ISO 3166-1)
- `Segment` is stored as uppercase (`PREMIUM`, `STANDARD`, `BASIC`)
- `SourceSystem` column is added (`CRM` or `Excel`) to track record origin

---

## Loading Layer

After transformation, data is written to two destinations simultaneously:

### Destination 1 -- Clean CSV (`data/clean/`)

The clean output CSV is written to Azure Blob Storage at the `data/clean/` path. This file can be downloaded and imported into Excel, Power BI, or any other tool without further cleaning.

**Output file:** `data/clean/customers_clean_YYYYMMDD.csv`

### Destination 2 -- SQL Server (`dbo.Customers`)

An **upsert (MERGE)** is performed against the SQL Server target table using the stored procedure `dbo.usp_UpsertCustomers`. This means:
- New customers are **inserted**
- Existing customers (matching `CustomerID`) are **updated**

The staging table `dbo.CustomerStaging` is used as an intermediate buffer and is truncated after each successful load.

---

## Data Storage

| Location | Type | Contents |
|---|---|---|
| `data/raw/` | Azure Blob Storage | Unmodified source files -- never changed by the pipeline |
| `data/clean/` | Azure Blob Storage | Clean output CSV files from each pipeline run |
| `dbo.Customers` | SQL Server table | Final customer records, upserted on each run |
| `dbo.CustomerStaging` | SQL Server table | Temporary buffer, cleared after each run |
| `dbo.ETLRunLog` | SQL Server table | Audit log of every pipeline run (rows loaded, status, timestamps) |

---

## Orchestration and Scheduling

The pipeline `pl_customer_etl` is managed entirely within **Azure Data Factory**.

### Manual Execution
- Click **Trigger Now** in ADF Studio to run the pipeline immediately
- Use **Debug** mode when testing changes (samples data instead of full files)

### Scheduled Execution (Recommended)
Create a **Schedule Trigger** in ADF Studio:
1. Go to **Manage > Triggers > New**
2. Set recurrence (e.g. weekly on Monday at 06:00 UTC)
3. Attach the trigger to `pl_customer_etl`
4. Publish changes

### Event-Based Trigger (Advanced)
For an automated workflow, set up a **Storage Event Trigger** that fires whenever a new file arrives in `data/raw/`. This removes the need for manual or scheduled runs.

---

## Logging and Monitoring

### ADF Monitor
All pipeline runs are visible in **ADF Studio > Monitor > Pipeline Runs**. For each run you can see:
- Status: Succeeded / Failed / In Progress
- Start time and duration
- Activity-level detail (which step failed and why)

### ETL Run Log Table
Each pipeline run is recorded in `dbo.ETLRunLog`:

```sql
SELECT * FROM dbo.ETLRunLog ORDER BY RunStart DESC;
```

This gives a history of every run with row counts and status.

### Alerting (Recommended)
Set up **Azure Monitor Alerts** on the ADF resource to send email notifications when:
- A pipeline run fails
- A pipeline takes longer than expected (duration threshold)

### Data Quality Monitoring
After each run, the validation queries in `sql/scripts/05_validation_queries.sql` should be executed to verify:
- Row counts match expectations
- No null key fields exist
- No duplicates were introduced

See the [Data Validation](Data-Validation) page for the full checklist.
