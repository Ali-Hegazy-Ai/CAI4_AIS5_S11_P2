# SQL Schema

This page documents the SQL Server schema used as the target for the ETL pipeline, including table definitions, views, and script conventions.

---

## Target Database

| Property | Value |
|---|---|
| Database name | `CustomerDW` (or as configured) |
| SQL Server version | SQL Server 2019 / Azure SQL Database |
| Default schema | `dbo` |

---

## Scripts Location

All SQL scripts are stored in `sql/scripts/`.

### Naming Convention

Scripts are numbered to enforce execution order:

| Script File | Purpose |
|---|---|
| `01_create_database.sql` | Create the target database (run once) |
| `02_create_tables.sql` | Create all target tables |
| `03_create_views.sql` | Create reporting views |
| `04_load_procedures.sql` | Stored procedures for upsert / incremental load |
| `05_validation_queries.sql` | Quality check queries |

---

## Tables

### `dbo.Customers`

Main fact/dimension table holding all cleaned customer records.

```sql
CREATE TABLE dbo.Customers (
    CustomerID      INT             NOT NULL PRIMARY KEY,
    CustomerName    NVARCHAR(200)   NOT NULL,
    Email           NVARCHAR(255)       NULL,
    Phone           NVARCHAR(50)        NULL,
    SignupDate      DATE                NULL,
    Country         NVARCHAR(100)       NULL,
    Segment         NVARCHAR(50)        NULL,
    SourceSystem    NVARCHAR(20)        NULL,  -- 'CRM' or 'Excel'
    LoadedAt        DATETIME2       NOT NULL DEFAULT GETUTCDATE()
);
```

### `dbo.CustomerStaging`

Staging table used by the ADF pipeline before the final upsert into `dbo.Customers`.

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
    LoadedAt        DATETIME2       NULL
);
```

### `dbo.ETLRunLog`

Audit log table to track each pipeline run.

```sql
CREATE TABLE dbo.ETLRunLog (
    RunID           INT             IDENTITY(1,1) PRIMARY KEY,
    PipelineName    NVARCHAR(100)   NOT NULL,
    RunStart        DATETIME2       NOT NULL,
    RunEnd          DATETIME2           NULL,
    RowsLoaded      INT                 NULL,
    RowsRejected    INT                 NULL,
    Status          NVARCHAR(20)        NULL,  -- 'Success', 'Failed', 'Running'
    Notes           NVARCHAR(MAX)       NULL
);
```

---

## Views

### `dbo.vw_CustomerSummary`

A summary view for reporting and BI consumption.

```sql
CREATE VIEW dbo.vw_CustomerSummary AS
SELECT
    CustomerID,
    CustomerName,
    Email,
    Country,
    Segment,
    SignupDate,
    YEAR(SignupDate)   AS SignupYear,
    MONTH(SignupDate)  AS SignupMonth,
    SourceSystem,
    LoadedAt
FROM dbo.Customers;
```

---

## Stored Procedures

### `dbo.usp_UpsertCustomers`

Merges staging data into the main `Customers` table (upsert pattern).

```sql
CREATE PROCEDURE dbo.usp_UpsertCustomers AS
BEGIN
    MERGE dbo.Customers AS target
    USING dbo.CustomerStaging AS source
        ON target.CustomerID = source.CustomerID
    WHEN MATCHED THEN
        UPDATE SET
            CustomerName = source.CustomerName,
            Email        = source.Email,
            Phone        = source.Phone,
            SignupDate   = source.SignupDate,
            Country      = source.Country,
            Segment      = source.Segment,
            SourceSystem = source.SourceSystem,
            LoadedAt     = GETUTCDATE()
    WHEN NOT MATCHED BY TARGET THEN
        INSERT (CustomerID, CustomerName, Email, Phone, SignupDate, Country, Segment, SourceSystem, LoadedAt)
        VALUES (source.CustomerID, source.CustomerName, source.Email, source.Phone,
                source.SignupDate, source.Country, source.Segment, source.SourceSystem, GETUTCDATE());

    TRUNCATE TABLE dbo.CustomerStaging;
END;
```

---

## Script Execution Order

Run scripts in this order against the target SQL Server:

```bash
1. 01_create_database.sql
2. 02_create_tables.sql
3. 03_create_views.sql
4. 04_load_procedures.sql
5. 05_validation_queries.sql   # run after first data load
```

---

## Adding New Scripts

1. Follow the numbered prefix convention.
2. Each script should be **idempotent** (safe to run more than once) — use `IF NOT EXISTS` or `CREATE OR ALTER` where possible.
3. Add a comment block at the top of each script:

```sql
-- ============================================================
-- Script:  03_create_views.sql
-- Purpose: Create reporting views
-- Author:  [Your Name]
-- Date:    YYYY-MM-DD
-- ============================================================
```
