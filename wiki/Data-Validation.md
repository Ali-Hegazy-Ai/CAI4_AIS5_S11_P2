# Data Validation

This page describes how to verify that the ETL pipeline produced correct, complete output after each run. Running these checks is mandatory before considering any pipeline run "done".

---

## Why Validation Matters

A pipeline that runs without errors does not automatically mean the output data is correct. The pipeline could:
- Drop rows silently due to a filter condition that is too strict
- Corrupt date values due to a wrong format string
- Produce duplicate rows if the deduplication logic has a bug
- Accept null values that should have been rejected

Validation queries catch these issues before they reach your analysts and reports.

---

## Validation Goals

| Goal | What It Checks |
|---|---|
| **Completeness** | Expected number of rows made it through |
| **Uniqueness** | No duplicate `CustomerID` values |
| **Accuracy** | Field values match expected formats and business rules |
| **Validity** | Dates are real dates, emails contain `@`, segments are in the allowed list |
| **Consistency** | CRM and Excel records merge without unexplained value conflicts |

---

## Validation Checklist

Run this checklist after every pipeline execution:

- [ ] Row count in `dbo.Customers` is non-zero
- [ ] Row count is within expected range (not massively lower than source)
- [ ] No NULL `CustomerID` values
- [ ] No NULL or blank `CustomerName` values
- [ ] No duplicate `CustomerID` values
- [ ] All `Segment` values are `PREMIUM`, `STANDARD`, or `BASIC`
- [ ] All non-null `SignupDate` values fall within `2000-01-01` to today
- [ ] All non-null `Email` values contain `@`
- [ ] Row counts from CRM and Excel sources are both represented in `SourceSystem`

---

## Validation SQL Queries

Save and run these from `sql/scripts/05_validation_queries.sql`.

### 1. Total Row Count

```sql
-- How many customers are in the table?
SELECT COUNT(*) AS TotalCustomers
FROM dbo.Customers;
```

**Expected:** Greater than 0. Compare against known source file row counts.

---

### 2. Null Primary Key

```sql
-- Rows with null CustomerID (should be 0 rows)
SELECT COUNT(*) AS NullCustomerID
FROM dbo.Customers
WHERE CustomerID IS NULL;
```

**Expected:** 0

---

### 3. Null or Blank Customer Name

```sql
-- Rows with null or blank CustomerName (should be 0 rows)
SELECT COUNT(*) AS BlankNames
FROM dbo.Customers
WHERE CustomerName IS NULL
   OR LTRIM(RTRIM(CustomerName)) = '';
```

**Expected:** 0

---

### 4. Duplicate CustomerID

```sql
-- CustomerIDs that appear more than once (should be 0 rows)
SELECT CustomerID, COUNT(*) AS Occurrences
FROM dbo.Customers
GROUP BY CustomerID
HAVING COUNT(*) > 1;
```

**Expected:** 0 rows returned

---

### 5. Invalid Email Format

```sql
-- Emails that don't look like email addresses
SELECT CustomerID, Email
FROM dbo.Customers
WHERE Email IS NOT NULL
  AND (Email NOT LIKE '%@%' OR Email NOT LIKE '%.%');
```

**Expected:** 0 rows (or small number with documented exceptions)

---

### 6. Unexpected Segment Values

```sql
-- Segment values outside the allowed list
SELECT DISTINCT Segment, COUNT(*) AS Count
FROM dbo.Customers
GROUP BY Segment
ORDER BY Count DESC;
```

**Expected:** Only `PREMIUM`, `STANDARD`, `BASIC`, or NULL (if allowed)

---

### 7. SignupDate Out of Range

```sql
-- Dates that are in the future or unrealistically old
SELECT CustomerID, SignupDate
FROM dbo.Customers
WHERE SignupDate IS NOT NULL
  AND (SignupDate < '2000-01-01' OR SignupDate > CAST(GETDATE() AS DATE));
```

**Expected:** 0 rows

---

### 8. Source System Distribution

```sql
-- How many records came from each source?
SELECT SourceSystem, COUNT(*) AS RecordCount
FROM dbo.Customers
GROUP BY SourceSystem;
```

**Expected:** Both `CRM` and `Excel` rows are present. If one is 0, the source file may not have been loaded.

---

### 9. Row Count by Country

```sql
-- Top countries by customer count
SELECT Country, COUNT(*) AS CustomerCount
FROM dbo.Customers
GROUP BY Country
ORDER BY CustomerCount DESC;
```

**Expected:** Countries are full English names (not ISO codes like `EGY`). No `NULL` country if source data was complete.

---

### 10. Latest Pipeline Run Status

```sql
-- What did the most recent pipeline run produce?
SELECT TOP 1
    PipelineName,
    RunStart,
    RunEnd,
    DATEDIFF(SECOND, RunStart, RunEnd) AS DurationSeconds,
    RowsLoaded,
    RowsRejected,
    Status,
    Notes
FROM dbo.ETLRunLog
ORDER BY RunStart DESC;
```

**Expected:** `Status = 'Success'`, `RowsLoaded > 0`, `RowsRejected` is documented.

---

## What To Do When a Check Fails

| Check That Failed | Likely Cause | Investigation Steps |
|---|---|---|
| Row count too low | Filter dropped too many rows | Review Filter step in Data Flow; check source file for unexpected nulls |
| Null CustomerID exists | Source file had blank IDs | Check source file; strengthen the Filter step |
| Duplicate CustomerID | Deduplication step not working | Review Aggregate step configuration in the Data Flow |
| Unexpected Segment value | Source file has new tier names | Add the new value to the allowed list or add a mapping |
| ISO country codes in output | Country mapping table incomplete | Update the country mapping in the Derived Column step |
| SourceSystem = NULL | `SourceSystem` column not set in Data Flow | Check the Derived Column step for the `SourceSystem` hardcoded value |

---

## Row Count Tracking Table

Update this table in `docs/project_flow.md` after each pipeline run:

| Run Date | CRM Raw Rows | Excel Raw Rows | Clean Output Rows | SQL Table Rows | Rejected Rows | Issues Found |
|---|---|---|---|---|---|---|
| YYYY-MM-DD | -- | -- | -- | -- | -- | None |

---

## Rejected Records

Rows that are filtered out by the pipeline (e.g. null `CustomerID`) should be captured in a rejected output location:

- **File path:** `data/rejected/rejected_YYYYMMDD.csv`
- **Columns:** All source columns + `RejectionReason`
- This allows the team to review and fix source data issues rather than silently losing records.

To implement this in ADF, add a second Sink in the Data Flow connected to the Filter step's "false" branch.
