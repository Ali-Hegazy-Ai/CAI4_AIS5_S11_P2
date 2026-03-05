# Data Validation

This page describes the data quality checks and validation approach used to confirm that the ETL pipeline produces correct, complete output.

---

## Validation Goals

1. **Completeness** — No expected rows are lost between raw input and clean output.
2. **Accuracy** — Transformed values are correct (types, formats, business rules).
3. **Uniqueness** — No duplicate `CustomerID` values in the final output.
4. **Consistency** — Data from CRM and Excel merges without conflicts.
5. **Conformity** — All columns match the expected schema (see [SQL Schema](SQL-Schema)).

---

## Validation Checklist (Run After Each Pipeline Execution)

| Check | Query / Method | Expected Result |
|---|---|---|
| Row count: raw CRM | Count rows in `data/raw/` CRM file | Matches source record |
| Row count: raw Excel | Count rows in `data/raw/` Excel file | Matches source record |
| Row count: clean output | Count rows in `data/clean/` output | ≤ sum of raw rows (dedup removes duplicates) |
| Row count: SQL table | `SELECT COUNT(*) FROM dbo.Customers` | Matches clean output count |
| Null CustomerID | See query below | 0 rows |
| Null CustomerName | See query below | 0 rows |
| Duplicate CustomerID | See query below | 0 rows |
| Invalid Email format | See query below | 0 rows (or documented exceptions) |
| Null SignupDate | See query below | Documented and acceptable |
| Segment values | See query below | Only `Premium`, `Standard`, `Basic` |

---

## Validation SQL Queries

Save these in `sql/scripts/05_validation_queries.sql`.

### 1. Row Count Check

```sql
-- Total rows in the Customers table
SELECT COUNT(*) AS TotalCustomers FROM dbo.Customers;
```

### 2. Null Key Fields

```sql
-- Customers with null CustomerID or CustomerName (should return 0 rows)
SELECT *
FROM dbo.Customers
WHERE CustomerID IS NULL
   OR CustomerName IS NULL OR LTRIM(RTRIM(CustomerName)) = '';
```

### 3. Duplicate CustomerID

```sql
-- Duplicate CustomerIDs (should return 0 rows after dedup)
SELECT CustomerID, COUNT(*) AS Occurrences
FROM dbo.Customers
GROUP BY CustomerID
HAVING COUNT(*) > 1;
```

### 4. Invalid Email Format

```sql
-- Emails that don't contain '@' (rough format check)
SELECT CustomerID, Email
FROM dbo.Customers
WHERE Email IS NOT NULL
  AND Email NOT LIKE '%@%.%';
```

### 5. Unexpected Segment Values

```sql
-- Segment values outside the expected list
SELECT DISTINCT Segment
FROM dbo.Customers
WHERE Segment NOT IN ('Premium', 'Standard', 'Basic');
```

### 6. SignupDate Range Check

```sql
-- Dates outside a reasonable range
SELECT CustomerID, SignupDate
FROM dbo.Customers
WHERE SignupDate < '2000-01-01'
   OR SignupDate > GETDATE();
```

### 7. Source System Distribution

```sql
-- How many records came from each source
SELECT SourceSystem, COUNT(*) AS RecordCount
FROM dbo.Customers
GROUP BY SourceSystem;
```

---

## Validation Workflow

1. After each pipeline run, open SSMS and run the queries above in `05_validation_queries.sql`.
2. Record results in a comment in the next commit message or in the `docs/project_flow.md` progress table.
3. If any check fails:
   - Identify the root cause in the source file or ADF transformation step.
   - Fix the data flow or SQL script.
   - Re-run the pipeline.
   - Re-run all validation checks.

---

## Row Count Tracking Template

Update this table in `docs/project_flow.md` after each cycle:

| Date | CRM Raw Rows | Excel Raw Rows | Clean Output Rows | SQL Table Rows | Issues Found |
|---|---|---|---|---|---|
| YYYY-MM-DD | — | — | — | — | None |

---

## Automated Validation (Future Improvement)

For a more advanced setup, consider:
- Adding a **Validation Activity** at the end of the ADF pipeline that runs `05_validation_queries.sql` and fails the run if any check returns rows.
- Using ADF's **Data Flow** debug statistics to capture row counts at each transformation step.
- Logging rejected rows to a `data/rejected/` folder with an error reason column.
