# Contributing

This page explains how to contribute to the project -- from setting up your local environment, to branching, committing, and opening pull requests.

> **Before you contribute:** Make sure you have completed the [Setup Guide](Setup-Guide) first.

---

## Contribution Workflow Overview

```
1. Pick an issue from GitHub Issues
       |
       v
2. Create a new branch from main
       |
       v
3. Make your changes locally
       |
       v
4. Test your changes
       |
       v
5. Commit and push your branch
       |
       v
6. Open a Pull Request on GitHub
       |
       v
7. Get a review from a teammate
       |
       v
8. Merge to main
```

---

## Step 1 -- Pick an Issue

All tasks, bugs, and feature requests are tracked as GitHub Issues. Before starting any work:

1. Go to the **Issues** tab in the repository.
2. Look for an unassigned open issue.
3. Comment on it: "I'll take this one" so no one else picks it up at the same time.
4. The issue will be assigned to you.

If something needs to be done but there is no issue for it:
- Open a new issue using one of the issue templates (Task, Bug Report, or Feature Request)
- Wait for a team member to confirm it before starting work

---

## Step 2 -- Create a Branch

Never work directly on the `main` branch. Always create a new branch for your work.

### Branch Naming Convention

| Type of Work | Branch Name Pattern | Example |
|---|---|---|
| New feature or task | `feature/short-description` | `feature/add-phone-normalisation` |
| Bug fix | `fix/short-description` | `fix/null-date-parse-error` |
| Documentation update | `docs/short-description` | `docs/update-sql-schema-page` |
| ADF pipeline changes | `pipeline/short-description` | `pipeline/add-excel-source` |

```bash
# Make sure you are on the latest main branch first
git checkout main
git pull origin main

# Create and switch to your new branch
git checkout -b feature/your-task-name
```

---

## Step 3 -- Make Your Changes

Make focused, small changes. A pull request should do **one thing** -- not five unrelated things.

### What Goes Where

| Change Type | Location |
|---|---|
| New or updated ADF pipeline | `adf/pipelines/` |
| New or updated ADF dataset | `adf/datasets/` |
| New or updated ADF linked service | `adf/linked_services/` |
| New SQL script | `sql/scripts/` (with correct number prefix) |
| Update to existing SQL script | New versioned script, not editing the old one |
| New or updated raw source file | `data/raw/` |
| Wiki update | `wiki/` |
| Project guide update | `docs/README.md` or `docs/project_flow.md` |

### Files You Should NOT Commit

- Files in `data/clean/` -- these are pipeline outputs, not source code
- Any credentials or connection strings
- Temporary files, `.DS_Store`, `Thumbs.db`, etc.

---

## Step 4 -- Test Your Changes

Before committing, verify your work:

| Change Type | How to Test |
|---|---|
| ADF pipeline change | Run in Debug mode in ADF Studio; check Monitor for errors |
| SQL script | Run in SSMS on a test database; verify tables/views/procedures were created |
| Source data file | Trigger a Debug pipeline run and check that the new file is read correctly |
| Documentation | Read it -- does it make sense to someone who does not already know the answer? |

---

## Step 5 -- Commit Your Changes

Write clear, descriptive commit messages:

```bash
# Stage your changes
git add .

# Commit with a descriptive message
git commit -m "Add phone number normalisation to Data Flow derived column step"

# Push your branch to GitHub
git push origin feature/your-task-name
```

### Good Commit Message Examples

- `Add 05_validation_queries.sql with 10 quality check queries`
- `Fix null SignupDate handling in df_merge_transform filter step`
- `Update ds_excel_source dataset to point to new sheet name`
- `Update SQL Schema wiki page with ETLRunLog table definition`

### Bad Commit Message Examples

- `fix` (too vague)
- `update files` (says nothing)
- `WIP` (not a complete change)

---

## Step 6 -- Open a Pull Request

1. Go to the repository on GitHub.
2. Click the **"Compare & pull request"** button that appears after you push a branch.
3. Fill in the pull request template:
   - Describe what you changed and why
   - Link to the issue: `Closes #<issue-number>`
   - Note any manual testing steps you completed

4. Assign a reviewer from the team.
5. Add labels (e.g. `pipeline`, `sql`, `documentation`).

---

## Step 7 -- Review Process

- Every pull request needs **at least one approval** from a teammate before merging.
- Address all review comments by either making changes or explaining why you disagree.
- If you push new commits to address review comments, re-request the review.
- Do not merge your own pull request without a review (except for documentation typos).

---

## Step 8 -- Merge

Once approved:
- The reviewer or author clicks **Squash and Merge** (preferred) to keep the `main` history clean.
- Delete the branch after merging.
- Close the linked issue if it was not closed automatically.

---

## Issue Templates

The repository has three issue templates available when you click **New Issue** on GitHub:

| Template | When to Use |
|---|---|
| **Task** | Any planned work item: feature, improvement, pipeline change |
| **Bug Report** | A data quality issue, pipeline failure, or incorrect output |
| **Feature Request** | A new capability or enhancement you want to propose |

---

## Labels

Use labels on issues and pull requests to make them easy to filter:

| Label | Meaning |
|---|---|
| `pipeline` | Related to ADF pipelines or Data Flows |
| `sql` | Related to SQL scripts or the database schema |
| `data-quality` | Related to data validation or quality issues |
| `documentation` | Wiki or docs updates |
| `bug` | Something is broken or producing wrong output |
| `enhancement` | Improvement to existing functionality |
| `blocked` | Work is blocked and waiting on another task or person |
| `good first issue` | Suitable for someone new to the project |

---

## Code Style Guidelines

### SQL

- Use `UPPER CASE` for SQL keywords: `SELECT`, `FROM`, `WHERE`, `JOIN`
- Use `snake_case` for column aliases in queries
- Prefix stored procedures with `usp_` (e.g. `dbo.usp_UpsertCustomers`)
- Add a standard comment header to every script file (see [SQL Schema](SQL-Schema))
- Scripts must be idempotent -- use `IF NOT EXISTS` or `CREATE OR ALTER`

### ADF JSON Files

- Never edit ADF JSON by hand -- always use ADF Studio and export
- Export the smallest possible scope (one pipeline, one dataset) to avoid large diffs

### File Names

- Use `snake_case` for source data files: `crm_customers_20240601.csv`
- Use `PascalCase` for SQL object names: `CustomerDW`, `dbo.Customers`
- Use numbered prefixes for SQL scripts: `01_`, `02_`, etc.
