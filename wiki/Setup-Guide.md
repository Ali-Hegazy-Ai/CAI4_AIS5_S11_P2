# Setup Guide

This guide walks you through everything you need to install and configure before you can work on this project. Follow the steps in order and you will be ready to run the pipeline or contribute code.

---

## Prerequisites

Install and configure the following before you begin:

| Tool | Minimum Version | What It Is Used For |
|---|---|---|
| Git | 2.x | Cloning the repo and version control |
| Azure CLI | Latest | Logging in to Azure from your terminal |
| ADF Studio | Browser-based | Designing and running the ADF pipeline |
| SQL Server Management Studio (SSMS) | 19.x | Running SQL scripts and querying the database |
| Microsoft Excel | 2016 / Microsoft 365 | Opening and inspecting source Excel files |
| Azure Subscription | Active | Required to use Azure Data Factory and Blob Storage |

### Download Links

- Git: https://git-scm.com/downloads
- Azure CLI: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli
- ADF Studio: https://adf.azure.com (browser, no install needed)
- SSMS: https://aka.ms/ssmsfullsetup

---

## Step 1 -- Clone the Repository

Open a terminal and run:

```bash
git clone https://github.com/Ali-Hegazy-Ai/customer-data-etl.git
cd customer-data-etl
```

You now have a local copy of the repository. The folder structure should look like:

```
customer-data-etl/
+-- data/raw/
+-- data/clean/
+-- sql/scripts/
+-- adf/
+-- docs/
+-- wiki/
```

---

## Step 2 -- Log In to Azure

Authenticate with the Azure CLI so you can access the Azure resources:

```bash
az login
```

A browser window will open. Sign in with your Azure account. When done, return to the terminal.

To verify you are logged in to the correct subscription:

```bash
az account show
```

If you need to switch subscriptions:

```bash
az account set --subscription "<your-subscription-id>"
```

---

## Step 3 -- Configure Azure Data Factory

1. Open a browser and go to: https://adf.azure.com
2. Select your Azure Subscription and the Data Factory resource named for this project.
3. Click **Launch Studio**.

### Import ADF Assets

The pipeline, dataset, and linked service definitions are stored as JSON files in the `adf/` folder. To import them:

1. In ADF Studio, go to **Author** (pencil icon on the left sidebar).
2. Click the three dots (`...`) next to **Pipelines** > **Import from file**.
3. Browse to `adf/pipelines/` and import each `.json` file.
4. Repeat for **Datasets** (import from `adf/datasets/`) and **Linked Services** (import from `adf/linked_services/`).

### Test Linked Service Connections

After importing:
1. Go to **Manage** > **Linked Services**.
2. Click each linked service and then click **Test Connection**.
3. If a connection fails, check the connection string credentials and firewall rules on the target resource.

---

## Step 4 -- Set Up the SQL Server Database

1. Open **SSMS** and connect to your SQL Server instance (get the server name and credentials from the team lead).
2. Create a new database called `CustomerDW` if it does not already exist.
3. Run the SQL scripts in `sql/scripts/` **in the numbered order**:

```text
01_create_database.sql    -- Creates the database (skip if already exists)
02_create_tables.sql      -- Creates all tables
03_create_views.sql       -- Creates reporting views
04_load_procedures.sql    -- Creates stored procedures for upsert logic
05_validation_queries.sql -- Run this after the first data load to validate
```

To run a script in SSMS:
- Open the file: **File > Open > File...**
- Ensure the correct database is selected in the top toolbar dropdown
- Click **Execute** (F5)

---

## Step 5 -- Add Source Data Files

Place source files in the `data/raw/` folder following the naming convention:

| Source | Naming Convention | Example |
|---|---|---|
| CRM Export | `crm_customers_YYYYMMDD.csv` | `crm_customers_20240601.csv` |
| Excel File | `excel_customers_YYYYMMDD.xlsx` | `excel_customers_20240601.xlsx` |

> **Important:** Never modify files in `data/raw/`. These are the source of truth. The pipeline reads them and writes output to `data/clean/`.

If you are using Azure Blob Storage, upload the files to the `raw/` container using:

```bash
az storage blob upload \
  --account-name <your-storage-account> \
  --container-name data \
  --name raw/crm_customers_20240601.csv \
  --file data/raw/crm_customers_20240601.csv
```

---

## Step 6 -- Run the Pipeline

1. In ADF Studio, go to **Author > Pipelines**.
2. Open `pl_customer_etl`.
3. Click **Debug** for a test run using a sample of data.
4. Review the output in **Monitor > Pipeline Runs**.
5. If the Debug run succeeds, click **Add Trigger > Trigger Now** for a full run.
6. After completion, verify output in `data/clean/` and run the validation queries in SSMS.

---

## Step 7 -- Set Up Your Git Workflow

See the [Contributing](Contributing) page for the full branching and pull request guide. Quick summary:

```bash
# Always work on a new branch, not main
git checkout -b feature/your-task-name

# After making changes, commit them
git add .
git commit -m "Short description of what you changed"

# Push and open a pull request on GitHub
git push origin feature/your-task-name
```

---

## Common Issues and Fixes

| Problem | Likely Cause | Fix |
|---|---|---|
| ADF linked service test connection fails | Wrong credentials or firewall | Update the connection string; add your IP to the SQL Server firewall |
| SQL script fails with "database not found" | Database not created yet | Run `01_create_database.sql` first |
| SQL script fails with "object already exists" | Script run twice | Scripts use `IF NOT EXISTS` -- check if object was partially created |
| Pipeline fails at Copy Excel activity | Wrong sheet name or file path | Verify `ds_excel_source` dataset points to correct file and sheet |
| `data/clean/` is empty after pipeline run | Sink configuration issue | Check the Sink dataset in the Data Flow points to `data/clean/` |
| Git push rejected | Local branch is behind remote | Run `git pull origin main` before pushing |
