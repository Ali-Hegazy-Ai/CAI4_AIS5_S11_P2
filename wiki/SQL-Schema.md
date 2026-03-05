# SQL Schema

This page documents the SQL Server database schema used as the target for the ETL pipeline -- including table definitions, views, stored procedures, and the script execution guide.

---

## Database

| Property | Value |
|---|---|
| Database name | `CustomerDW` |
| SQL Server version | SQL Server 2019 or Azure SQL Database |
| Default schema | `dbo` |
| Collation | SQL_Latin1_General_CP1_CI_AS (default) |

---

## Scripts Location and Naming Convention

All SQL scripts live in `sql/scripts/`. Scripts are numbered to enforce a specific execution order:

| Script | Purpose |
|---|---|
| `01_create_database.sql` | Creates the `CustomerDW` database (run once, skip if it exists) |
| `02_create_tables.sql` | Creates all tables (`Customers`, `CustomerStaging`, `ETLRunLog`) |
| `03_create_views.sql` | Creates reporting views |
| `04_load_procedures.sql` | Creates stored procedures for upsert and data loading |
| `05_validation_queries.sql` | Quality-check queries -- run after each pipeline execution |

> All scripts are written to be **idempotent** -- safe to run more than once. They use `IF NOT EXISTS` or `CREATE OR ALTER` patterns so they will not fail if the object already exists.

### Script Header Convention

Each script starts with a standard comment block:

```sql
-- ============================================================
-- Script:  02_create_tables.sql
-- Purpose: Create all target tables for the CustomerDW database
-- Author:  [Your Name]
-- Date:    YYYY-MM-DD
-- Version: 1.0
-- ============================================================
```

---

## Tables

### dbo.Customers

The main table holding all final, clean customer records after the ETL pipeline runs.

```sql
CREATE TABLE dbo.Customers (
    CustomerID      INT             NOT NULL,
    CustomerName    NVARCHAR(200)   NOT NULL,
    Email           NVARCHAR(255)       NULL,
    Phone           NVARCHAR(50)        NULL,
    SignupDate      DATE                NULL,
    Country         NVARCHAR(100)       NULL,
    Segment         NVARCHAR(50)        NULL,
    SourceSystem    NVARCHAR(20)        NULL,
    LoadedAt        DATETIME2(0)    NOT NULL DEFAULT GETUTCDATE(),

    CONSTRAINT PK_Customers PRIMARY KEY (CustomerID)
);
```

**Column Descriptions:**

| Column | Type | Nullable | Description |
|---|---|---|---|
| `CustomerID` | INT | NOT NULL | Primary key -- unique customer identifier |
| `CustomerName` | NVARCHAR(200) | NOT NULL | Full name, title-cased and trimmed |
| `Email` | NVARCHAR(255) | NULL | Email address, lowercased |
| `Phone` | NVARCHAR(50) | NULL | Normalised phone number |
| `SignupDate` | DATE | NULL | Date customer registered (NULL allowed for old records) |
| `Country` | NVARCHAR(100) | NULL | Full English country name |
| `Segment` | NVARCHAR(50) | NULL | Customer tier: `PREMIUM`, `STANDARD`, or `BASIC` |
| `SourceSystem` | NVARCHAR(20) | NULL | Origin of the record: `CRM` or `Excel` |
| `LoadedAt` | DATETIME2 | NOT NULL | UTC timestamp of when the row was last loaded |

---

### dbo.CustomerStaging

A temporary buffer table used by the ADF pipeline. The pipeline loads transformed data here first, then the `usp_UpsertCustomers` procedure merges it into `dbo.Customers`.

```sql
CREATE TABLE dbo.CustomerStaging (
    CustomerID      INT             NULL,
    CustomerName    NVARCHAR(200)   NULL,
    Email           NVARCHAR(255)   NULL,
    Phone           NVARCHAR(50)    NULL,
    SignupDate      DATE            NULL,
    Country         NVARCHAR(100)   NULL,
    Segment         NVARCHAR(50)    NULL,
    SourceSystem    NVARCHAR(20)    NULL,
    LoadedAt        DATETIME2(0)    NULL
);
```

> The staging table has no primary key and allows NULLs -- this is intentional so that ADF can bulk-load data without constraint errors during staging.

---

### dbo.ETLRunLog

An audit log that records the outcome of every pipeline run.

```sql
CREATE TABLE dbo.ETLRunLog (
    RunID           INT             NOT NULL IDENTITY(1,1),
    PipelineName    NVARCHAR(100)   NOT NULL,
    RunStart        DATETIME2(0)    NOT NULL,
    RunEnd          DATETIME2(0)        NULL,
    RowsLoaded      INT                 NULL,
    RowsRejected    INT                 NULL,
    Status          NVARCHAR(20)        NULL,
    Notes           NVARCHAR(MAX)       NULL,

    CONSTRAINT PK_ETLRunLog PRIMARY KEY (RunID)
);
```

`Status` values: `Running`, `Success`, `Failed`

---

## Views

### dbo.vw_CustomerSummary

A simplified reporting view that exposes the most useful customer fields along with derived year/month columns for easy time-based analysis.

```sql
CREATE OR ALTER VIEW dbo.vw_CustomerSummary AS
SELECT
    CustomerID,
    CustomerName,
    Email,
    Country,
    Segment,
    SignupDate,
    YEAR(SignupDate)    AS SignupYear,
    MONTH(SignupDate)   AS SignupMonth,
    SourceSystem,
    LoadedAt
FROM dbo.Customers;
```

---

## Stored Procedures

### dbo.usp_UpsertCustomers

Performs an upsert (MERGE) from the staging table into `dbo.Customers`, then clears the staging table.

```sql
CREATE OR ALTER PROCEDURE dbo.usp_UpsertCustomers AS
BEGIN
    SET NOCOUNT ON;

    MERGE dbo.Customers AS target
    USING dbo.CustomerStaging AS source
        ON target.CustomerID = source.CustomerID
    WHEN MATCHED THEN
        UPDATE SET
            CustomerName  = source.CustomerName,
            Email         = source.Email,
            Phone         = source.Phone,
            SignupDate    = source.SignupDate,
            Country       = source.Country,
            Segment       = source.Segment,
            SourceSystem  = source.SourceSystem,
            LoadedAt      = GETUTCDATE()
    WHEN NOT MATCHED BY TARGET THEN
        INSERT (CustomerID, CustomerName, Email, Phone,
                SignupDate, Country, Segment, SourceSystem, LoadedAt)
        VALUES (source.CustomerID, source.CustomerName, source.Email, source.Phone,
                source.SignupDate, source.Country, source.Segment,
                source.SourceSystem, GETUTCDATE());

    TRUNCATE TABLE dbo.CustomerStaging;
END;
```

---

## How to Run the Scripts

### First-Time Setup (run once)

Open SSMS and run scripts in order:

```
1. 01_create_database.sql   -- creates CustomerDW
2. 02_create_tables.sql     -- creates all tables
3. 03_create_views.sql      -- creates views
4. 04_load_procedures.sql   -- creates stored procedures
```

After the first pipeline run:

```
5. 05_validation_queries.sql  -- check data quality
```

### How to Run a Script in SSMS

1. Open SSMS and connect to your SQL Server.
2. Go to **File > Open > File...** and select the script.
3. In the toolbar, select the correct database from the dropdown (e.g. `CustomerDW`).
4. Press **F5** or click **Execute**.

---

## Useful Ad-Hoc Queries

```sql
-- How many customers are loaded?
SELECT COUNT(*) AS TotalCustomers FROM dbo.Customers;

-- How many customers per segment?
SELECT Segment, COUNT(*) AS Count
FROM dbo.Customers
GROUP BY Segment
ORDER BY Count DESC;

-- Most recent pipeline run result
SELECT TOP 1 * FROM dbo.ETLRunLog ORDER BY RunStart DESC;

-- Customers added in the last 30 days
SELECT * FROM dbo.Customers WHERE LoadedAt >= DATEADD(DAY, -30, GETUTCDATE());
```

---

## Adding or Modifying Schema

When you need to add a new column, table, or procedure:

1. Create a new numbered script in `sql/scripts/` (e.g. `06_add_region_column.sql`).
2. Use `ALTER TABLE` or `CREATE OR ALTER PROCEDURE` -- do not modify earlier scripts.
3. Test the script locally in SSMS before committing.
4. Commit and push the new script file.
5. Ask the team to run the new script on any shared environment.
