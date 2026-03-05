# ETL Pipeline

This page describes the Azure Data Factory pipeline architecture, data flow steps, and transformation logic used in this project.

---

## Pipeline Overview

The pipeline ingests customer data from two sources, merges and transforms it, and writes clean output to the `data/clean/` folder and/or a SQL Server target table.

```
[CRM Source File]  ──┐
                      ├──► [ADF Data Flow: Merge + Transform] ──► [data/clean/] ──► [SQL Server Table]
[Excel Source File] ──┘
```

---

## ADF Assets

All ADF assets are exported as JSON and stored in the repository:

| Folder | Contents |
|---|---|
| `adf/pipelines/` | Pipeline definitions (orchestration logic) |
| `adf/datasets/` | Dataset definitions (source and sink schemas) |
| `adf/linked_services/` | Connection definitions (storage accounts, SQL Server) |

After making changes in ADF Studio, always export and commit the updated JSON files.

---

## Linked Services

| Name | Type | Purpose |
|---|---|---|
| `ls_blob_storage` | Azure Blob Storage | Connects to the storage container holding `data/raw/` and `data/clean/` |
| `ls_sql_server` | SQL Server | Connects to the target SQL Server database |

---

## Datasets

| Name | Type | Linked Service | Notes |
|---|---|---|---|
| `ds_crm_source` | DelimitedText (CSV) | `ls_blob_storage` | Points to `data/raw/` CRM file |
| `ds_excel_source` | Excel | `ls_blob_storage` | Points to `data/raw/` Excel file |
| `ds_clean_output` | DelimitedText (CSV) | `ls_blob_storage` | Points to `data/clean/` output |
| `ds_sql_customers` | SQL Server Table | `ls_sql_server` | Target table for cleaned data |

---

## Pipeline: `pl_customer_etl`

### Activities (in order)

1. **Copy CRM Data** — Copy activity reads the CRM CSV from `data/raw/` into a staging area.
2. **Copy Excel Data** — Copy activity reads the Excel file from `data/raw/` into a staging area.
3. **Data Flow: Transform & Merge** — Data Flow activity merges both sources and applies transformation rules.
4. **Copy to Clean Output** — Copy activity writes the transformed data to `data/clean/`.
5. **Load to SQL** — Copy activity loads the clean output into the SQL Server target table.

---

## Data Flow: `df_merge_transform`

### Transformation Steps

| Step | Transformation | Description |
|---|---|---|
| 1 | **Source (CRM)** | Read CRM customer rows |
| 2 | **Source (Excel)** | Read Excel customer rows |
| 3 | **Select** | Standardize column names across both sources |
| 4 | **Derived Column** | Standardize data types (dates, phone formats, etc.) |
| 5 | **Filter** | Remove rows with null CustomerID or CustomerName |
| 6 | **Aggregate / Exists** | Deduplicate on CustomerID — keep the most recent record |
| 7 | **Union** | Merge CRM and Excel streams into one |
| 8 | **Sink (CSV)** | Write to `data/clean/` |
| 9 | **Sink (SQL)** | Upsert into the SQL Server target table |

---

## Column Mapping

| Source Column (CRM / Excel) | Target Column | Type | Transformation Notes |
|---|---|---|---|
| `customer_id` / `CustomerID` | `CustomerID` | INT | Cast to integer, reject nulls |
| `full_name` / `Name` | `CustomerName` | NVARCHAR(200) | Trim whitespace, title case |
| `email` / `EmailAddress` | `Email` | NVARCHAR(255) | Lowercase, validate format |
| `phone` / `PhoneNumber` | `Phone` | NVARCHAR(50) | Normalize to `+XX-XXX-XXXXXXX` |
| `signup_date` / `JoinDate` | `SignupDate` | DATE | Parse to ISO format `YYYY-MM-DD` |
| `country` / `Country` | `Country` | NVARCHAR(100) | Standardize to ISO 3166-1 name |
| `segment` / `CustomerSegment` | `Segment` | NVARCHAR(50) | Uppercase |

---

## Running the Pipeline

1. Open [ADF Studio](https://adf.azure.com) and navigate to **Author → Pipelines**.
2. Open `pl_customer_etl`.
3. Click **Debug** to run with current inputs, or **Add Trigger → Trigger Now** for a manual full run.
4. Monitor progress in **Monitor → Pipeline Runs**.
5. After successful completion, verify output files in `data/clean/` and row counts in the SQL table.

---

## Updating ADF Assets

After any change in ADF Studio:
1. Export the affected pipeline/dataset/linked service as JSON.
2. Replace the corresponding file in `adf/pipelines/`, `adf/datasets/`, or `adf/linked_services/`.
3. Commit and push the updated file.

```bash
git add adf/
git commit -m "Update ADF pipeline: <description of change>"
git push
```
