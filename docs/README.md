# ETL for Customer Data from Multiple Sources



## Project Goal
Merge customer data from CRM and Excel, clean/transform it, and load clean output that is ready for Data Warehouse analysis.

## Tech Stack
- Azure Data Factory
- SQL Server
- Data Flows

## Project Folder Map

Use only these folders:

```
customer-data-etl/
├── data/
│   ├── raw/      # Original source files (CRM export, Excel)
│   └── clean/    # Final cleaned files ready for analysis
├── sql/
│   └── scripts/  # SQL scripts for tables/views/load steps
├── adf/
│   ├── pipelines/        # ADF pipelines
│   ├── datasets/         # ADF datasets
│   └── linked_services/  # ADF connections
└── docs/
		├── README.md         # This file
		└── project_flow.md   # Team phases and workflow
```

## Quick Working Rules (Student Version)
1. Put source input files in `data/raw/`.
2. Keep transformed output in `data/clean/`.
3. Save every SQL change as a new script in `sql/scripts/`.
4. Save/export ADF JSON assets into `adf/` folders.
5. Update `docs/project_flow.md` after each phase milestone.

## Team Responsibilities

Team members: **Ali, Amin, Mennat Allah, Aseel, Habiba**

Each role below is intentionally unassigned first so team members can claim ownership.

### 1) Data Collection / Extraction
- Owner: **[Unassigned - Claim this role]**
- Collect CRM and Excel source files.
- Validate file names and columns.
- Place files in `data/raw/`.

### 2) Data Cleaning / Transformation (ETL)
- Owner: **[Unassigned - Claim this role]**
- Build ADF pipelines and data flows.
- Standardize columns, types, null handling, and duplicates.
- Produce cleaned output in `data/clean/`.
- Store ADF assets in:
	- `adf/pipelines/`
	- `adf/datasets/`
	- `adf/linked_services/`

### 3) Data Modeling / Warehouse Design
- Owner: **[Unassigned - Claim this role]**
- Design warehouse tables (dimension/fact or simple reporting tables).
- Create SQL scripts in `sql/scripts/`.
- Keep naming consistent and readable.

### 4) Data Validation / Testing
- Owner: **[Unassigned - Claim this role]**
- Compare row counts: raw vs clean.
- Check nulls, duplicates, and key consistency.
- Confirm transformed values are correct.
- Record quick validation notes in commit messages or phase notes.

### 5) Documentation & Presentation
- Owner: **[Unassigned - Claim this role]**
- Keep project progress updated in `docs/project_flow.md`.
- Prepare final report slides/screenshots.
- Explain source → transform → output clearly.

## Simple Weekly Workflow
1. Add/update source files in `data/raw/`.
2. Update ADF assets in `adf/`.
3. Update SQL in `sql/scripts/`.
4. Run ETL and generate output in `data/clean/`.
5. Validate output and update `docs/project_flow.md`.

## Expected Output
Clean customer dataset ready for analysis in Data Warehouse/BI reporting.

