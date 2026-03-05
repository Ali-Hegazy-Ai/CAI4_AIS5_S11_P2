# Project Workflow Guide (DEPI Final Project - Iterative)

## ETL for Customer Data from Multiple Sources

This project follows a practical iterative workflow for a 4-week DEPI final project.
The team works in overlapping cycles so each cycle improves the pipeline quality and output.

---

## Team Roles and Assignments (Working in Parallel)

Team members: **Ali, Amin, Mennat Allah, Aseel, Habiba**

Each role is intentionally marked as unassigned first, so teammates can claim ownership.

- **Role 1 — Data Collection & Cloud Storage (Extraction)**
	- Owner: **[Unassigned - Claim this role]**
	- Collect and profile CRM and Excel source files.
	- Organize source files in `data/raw/`.
	- Prepare storage structure for raw/bronze layer.

- **Role 2 — Data Cleaning & Orchestration (ETL)**
	- Owner: **[Unassigned - Claim this role]**
	- Build ADF linked services, datasets, and pipelines.
	- Implement Data Flows to merge CRM + Excel, handle nulls, standardize types, and deduplicate.
	- Move processed output to `data/clean/`.

- **Role 3 — Data Modeling & SQL Architecture**
	- Owner: **[Unassigned - Claim this role]**
	- Design target warehouse schema.
	- Maintain table/load scripts in `sql/scripts/`.
	- Add SQL procedures for update/upsert behavior.

- **Role 4 — Data Quality & Automated Testing**
	- Owner: **[Unassigned - Claim this role]**
	- Define data quality checks (nulls, duplicates, row counts).
	- Create validation SQL scripts and rejected/bad-row logging approach.
	- Add basic pipeline error-handling checks.

- **Role 5 — BI Analytics & Documentation**
	- Owner: **[Unassigned - Claim this role]**
	- Document lineage and progress in `docs/README.md` and `docs/project_flow.md`.
	- Prepare final reporting story and dashboard handoff notes.

---

## Iterative Delivery Cycles (4 Weeks)

## Cycle 1 (Week 1): Minimal Viable Pipeline

### Goal
Deliver the first working version using sample CRM + Excel data.

### Parallel Work
- Extraction role places first sample files in `data/raw/`.
- ETL role builds first ADF path raw → clean using `adf/` assets.
- Modeling role drafts initial SQL scripts in `sql/scripts/`.
- Testing role validates output quality immediately.
- Documentation role captures setup and first run notes in `docs/`.

### Expected Output
- First successful run producing cleaned output in `data/clean/`.

---

## Cycle 2 (Week 2-3): Full Data Load + Automated Transformations

### Goal
Scale from sample data to full data and strengthen transformation rules.

### Parallel Work
- Extraction role updates `data/raw/` with fuller files.
- ETL role automates transformation rules and orchestration.
- Modeling role updates warehouse scripts and SQL logic.
- Testing role runs repeated checks after each iteration.
- Documentation role records issues, decisions, and fixes.

### Expected Output
- Stable pipeline with automated transformations and cleaner, consistent outputs.

---

## Cycle 3 (Week 4): Data Validation + Final Presentation

### Goal
Finalize quality, freeze outputs, and prepare final DEPI presentation.

### Parallel Work
- Extraction role performs final source refresh.
- ETL role finalizes pipeline behavior and run sequence.
- Modeling role finalizes SQL scripts and procedure logic.
- Testing role executes final validation checklist and quality sign-off.
- Documentation role prepares final portfolio narrative and presentation.

### Expected Output
- Final clean data in `data/clean/`.
- Final ADF assets and SQL scripts.
- Portfolio-ready final project documentation.

---

## Folder Usage Reference

- `data/raw/` → source CRM and Excel files
- `data/clean/` → transformed output ready for analytics
- `adf/pipelines/` → orchestration pipelines
- `adf/datasets/` → source/target datasets
- `adf/linked_services/` → service connections
- `sql/scripts/` → schema + transformation/load scripts
- `docs/README.md` + `docs/project_flow.md` → documentation, status, and final narrative

---

## Iterative Progress Tracker

Update this table at least once per week:

| Cycle | Focus | Extraction Owner | ETL Owner | Modeling Owner | Testing Owner | Docs Owner | Cycle Status |
|---|---|---|---|---|---|---|---|
| Cycle 1 | MVP pipeline with sample data | Unassigned / Name | Unassigned / Name | Unassigned / Name | Unassigned / Name | Unassigned / Name | Planned / Active / Complete |
| Cycle 2 | Full load + automated transformations | Unassigned / Name | Unassigned / Name | Unassigned / Name | Unassigned / Name | Unassigned / Name | Planned / Active / Complete |
| Cycle 3 | Data validation + final presentation | Unassigned / Name | Unassigned / Name | Unassigned / Name | Unassigned / Name | Unassigned / Name | Planned / Active / Complete |


