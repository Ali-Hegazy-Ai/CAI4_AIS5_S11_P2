# Setup Guide

This page walks every team member through cloning the repository and configuring the Azure tools needed to work on the ETL pipeline.

---

## 1. Prerequisites

Make sure you have the following installed/configured before you start:

| Tool | Minimum Version | Download |
|---|---|---|
| Git | 2.x | https://git-scm.com |
| Azure CLI | Latest | https://aka.ms/installazurecliwindows |
| Azure Data Factory Studio | (browser-based) | https://adf.azure.com |
| SQL Server Management Studio (SSMS) | 19.x | https://aka.ms/ssmsfullsetup |
| Microsoft Excel | 2016+ or 365 | Microsoft 365 |

An active **Azure subscription** is required to use Azure Data Factory.

---

## 2. Clone the Repository

```bash
git clone https://github.com/Ali-Hegazy-Ai/customer-data-etl.git
cd customer-data-etl
```

---

## 3. Repository Folder Conventions

| Folder | Purpose |
|---|---|
| `data/raw/` | Drop source CRM and Excel files here — **never edit these** |
| `data/clean/` | Cleaned output files produced by the pipeline |
| `sql/scripts/` | All SQL schema and load scripts |
| `adf/pipelines/` | Exported ADF pipeline JSON files |
| `adf/datasets/` | Exported ADF dataset JSON files |
| `adf/linked_services/` | Exported ADF linked service JSON files |
| `docs/` | Project documentation |
| `wiki/` | Extended wiki pages |

---

## 4. Azure Data Factory Setup

1. Log in to the Azure Portal: https://portal.azure.com
2. Navigate to your resource group and open the **Azure Data Factory** instance.
3. Click **Launch Studio** to open ADF Studio.
4. Import the pipeline definitions from `adf/pipelines/` using the ADF Studio import/export feature.
5. Import datasets from `adf/datasets/` and linked services from `adf/linked_services/`.
6. Verify all linked service connections by clicking **Test Connection**.

---

## 5. SQL Server Setup

1. Open **SSMS** and connect to your SQL Server instance.
2. Run the scripts in `sql/scripts/` **in order** (scripts are prefixed with numbers, e.g. `01_create_tables.sql`, `02_create_views.sql`).
3. Verify that all tables and views are created without errors.

---

## 6. Working with Source Data

1. Place original CRM export files and Excel files into `data/raw/`.
2. **Do not modify raw files** — they are the source of truth.
3. After running the pipeline, cleaned output will appear in `data/clean/`.

---

## 7. Branching and Committing

Follow this simple workflow:

```bash
# Create a branch for your task
git checkout -b feature/your-task-name

# Stage and commit your changes
git add .
git commit -m "Short description of what you did"

# Push and open a pull request
git push origin feature/your-task-name
```

See [CONTRIBUTING.md](../CONTRIBUTING.md) for the full contribution guide.

---

## 8. Common Issues

| Problem | Solution |
|---|---|
| ADF linked service connection fails | Check connection string credentials and firewall rules |
| SQL script errors on first run | Make sure you run scripts in order and the database exists |
| Raw file not picked up by pipeline | Verify file name matches the expected pattern in the ADF dataset |
| Git push rejected | Pull latest changes first: `git pull origin main` |
