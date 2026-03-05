---
name: Bug Report
about: Report a pipeline failure, data quality issue, or incorrect output
title: "[BUG] Short description of the bug"
labels: bug
assignees: ''
---

## What Happened?

<!-- Describe the bug clearly. What did you observe that was wrong? -->

## What Did You Expect to Happen?

<!-- Describe what the correct behavior or output should be. -->

## Steps to Reproduce

1. 
2. 
3. 

## Where Was the Bug Found?

<!-- Check all that apply -->

- [ ] ADF Pipeline (activity failure or wrong row count)
- [ ] ADF Data Flow (transformation produced wrong values)
- [ ] SQL script (script failed or produced wrong schema)
- [ ] Source data file (unexpected format or missing data)
- [ ] Validation query (check returned unexpected results)
- [ ] Documentation (incorrect or misleading content)

## Error Details

<!-- Paste any error messages, ADF activity output, or SQL error text here. -->

```
paste error here
```

## Pipeline Run Details (if applicable)

| Field | Value |
|---|---|
| Pipeline name | pl_customer_etl |
| Run ID (from ADF Monitor) | |
| Activity that failed | |
| Run date | YYYY-MM-DD |
| Source files used | |

## Validation Query Results (if applicable)

<!-- Paste relevant query output here. -->

```sql
-- Query and result
```

## Possible Cause

<!-- If you have an idea of what caused this, describe it here. -->

## Priority

- [ ] Critical -- pipeline is completely broken or data loss occurred
- [ ] High -- significant incorrect data in output
- [ ] Medium -- minor incorrect values or formatting issue
- [ ] Low -- cosmetic or documentation issue
