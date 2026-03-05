## Summary

<!-- Describe what this PR does in 1-3 sentences. -->

## Related Issue

Closes #<!-- issue number -->

## Type of Change

- [ ] Task / new feature
- [ ] Bug fix
- [ ] ADF pipeline or dataset change
- [ ] SQL script (new or updated)
- [ ] Documentation / wiki update
- [ ] Source data file

## Changes Made

<!-- List the files changed and what was changed in each. -->

| File | What Changed |
|---|---|
| | |
| | |

## Testing Done

<!-- Describe how you tested your changes before opening this PR. -->

- [ ] ADF pipeline Debug run completed successfully
- [ ] SQL script executed in SSMS without errors
- [ ] Validation queries run and all checks passed
- [ ] Row counts match expectations
- [ ] Manually reviewed output in `data/clean/` or queried `dbo.Customers`

## Validation Query Results (if applicable)

<!-- Paste a summary of validation results here. -->

| Check | Result |
|---|---|
| Null CustomerID | 0 rows |
| Duplicate CustomerID | 0 rows |
| Total rows loaded | |
| Rows rejected | |

## ADF Changes (if applicable)

- [ ] Updated JSON files exported and committed to `adf/`
- [ ] Linked service test connections verified
- [ ] Pipeline published in ADF Studio before export

## SQL Changes (if applicable)

- [ ] Script follows naming convention (numbered prefix)
- [ ] Script is idempotent (safe to run more than once)
- [ ] Script header comment block included

## Checklist

- [ ] My changes follow the branching and commit conventions in the [Contributing guide](wiki/Contributing.md)
- [ ] I have tested my changes
- [ ] I have updated relevant wiki or docs pages if needed
- [ ] I have linked this PR to the relevant GitHub Issue
