# Contributing to Customer Data ETL

Thank you for contributing to this project! This file contains the quick-start rules. For the full guide, see the [Contributing wiki page](wiki/Contributing.md).

---

## Quick Rules

1. **Always work on a branch** -- never commit directly to `main`
2. **One PR per task** -- keep pull requests focused on a single change
3. **Link every PR to an issue** -- use `Closes #<issue-number>` in the PR description
4. **Test before you push** -- run the ADF Debug pipeline or SQL script before opening a PR
5. **Review before merging** -- every PR needs at least one teammate approval

---

## Branch Naming

| Type | Pattern | Example |
|---|---|---|
| Task / feature | `feature/short-description` | `feature/add-phone-normalisation` |
| Bug fix | `fix/short-description` | `fix/null-date-error` |
| Documentation | `docs/short-description` | `docs/update-glossary` |
| Pipeline change | `pipeline/short-description` | `pipeline/add-excel-dedup` |

---

## Quick Workflow

```bash
# Start from the latest main
git checkout main
git pull origin main

# Create your branch
git checkout -b feature/your-task-name

# ... make changes and test ...

# Commit your work
git add .
git commit -m "Describe what you changed"

# Push and open a PR
git push origin feature/your-task-name
```

---

## Where Things Go

| Change | Folder / File |
|---|---|
| ADF pipeline JSON | `adf/pipelines/` |
| ADF dataset JSON | `adf/datasets/` |
| ADF linked service JSON | `adf/linked_services/` |
| New SQL scripts | `sql/scripts/` (numbered) |
| Raw source files | `data/raw/` |
| Wiki pages | `wiki/` |
| Project docs | `docs/` |

**Do NOT commit** files from `data/clean/` (pipeline output), credentials, or temporary files.

---

## Issue Templates

When opening a new issue, choose the appropriate template:

- **Task** -- planned work items
- **Bug Report** -- pipeline failures or wrong output
- **Feature Request** -- new capabilities or improvements

---

For the full guide including code style rules, label definitions, and the review process, see [wiki/Contributing.md](wiki/Contributing.md).
