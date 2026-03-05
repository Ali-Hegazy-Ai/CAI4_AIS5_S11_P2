# Team Roles

This page documents who owns what in the project, what each role is responsible for, and how to claim a role.

---

## Team Members

| Name | GitHub Handle | Role |
|---|---|---|
| Ali | @Ali-Hegazy-Ai | Project Lead / Architecture |
| Amin | TBD | ETL / ADF Pipeline |
| Mennat Allah | TBD | Data Modeling / SQL |
| Aseel | TBD | Data Validation / Testing |
| Habiba | TBD | Documentation / BI |

---

## How to Claim a Role

1. Open the `docs/project_flow.md` file.
2. Replace `[Unassigned - Claim this role]` with your name in the relevant role row.
3. Commit and push on a new branch, then open a pull request.
4. Update your row in the team table at the top of this page.

---

## Role Descriptions

### Role 1 -- Data Collection and Extraction

**Owner:** [Unassigned]

**Responsibilities:**
- Coordinate with the CRM team to obtain the latest customer export (CSV)
- Coordinate with the sales team to get the latest Excel customer file
- Validate that source files have the expected column names and formats (see [Data Sources](Data-Sources))
- Name files following the convention: `crm_customers_YYYYMMDD.csv` and `excel_customers_YYYYMMDD.xlsx`
- Place files in `data/raw/` before each pipeline run
- Report any schema changes (new/renamed columns) to the ETL role owner immediately

**Weekly Task Checklist:**
- [ ] New CRM file obtained and placed in `data/raw/`
- [ ] New Excel file obtained and placed in `data/raw/`
- [ ] File schemas verified (no new/missing columns)
- [ ] Files committed to the repository

---

### Role 2 -- ETL and ADF Pipeline

**Owner:** [Unassigned]

**Responsibilities:**
- Build and maintain ADF linked services, datasets, and pipelines
- Implement Data Flow transformations (column renaming, type casting, deduplication)
- Handle schema changes when new columns appear in source files
- Ensure the pipeline runs successfully end-to-end
- Export updated ADF assets as JSON and commit them to `adf/`
- Write pipeline run notes in `docs/project_flow.md`

**Weekly Task Checklist:**
- [ ] Pipeline runs successfully in Debug mode on new source files
- [ ] Pipeline runs successfully in full (Trigger Now) mode
- [ ] Updated ADF JSON files committed to `adf/`
- [ ] Pipeline run result noted in `docs/project_flow.md`

---

### Role 3 -- Data Modeling and SQL

**Owner:** [Unassigned]

**Responsibilities:**
- Design and maintain the SQL Server target schema (`dbo.Customers`, views, procedures)
- Write new SQL scripts for schema changes and save them in `sql/scripts/`
- Follow the numbered script naming convention and the idempotency rule
- Maintain the stored procedure `dbo.usp_UpsertCustomers`
- Support the ETL role if loading issues are related to the SQL schema

**Weekly Task Checklist:**
- [ ] Any new SQL scripts committed to `sql/scripts/`
- [ ] Scripts tested in SSMS before committing
- [ ] Schema changes documented in the SQL Schema wiki page

---

### Role 4 -- Data Validation and Testing

**Owner:** [Unassigned]

**Responsibilities:**
- Run the validation checklist in `sql/scripts/05_validation_queries.sql` after every pipeline run
- Compare row counts: raw source vs clean output vs SQL table
- Check for nulls, duplicates, invalid formats, and out-of-range values
- Document any data quality issues found and report them to the relevant role owner
- Update the row count tracking table in `docs/project_flow.md`
- (Advanced) Implement a rejected records capture to `data/rejected/`

**Weekly Task Checklist:**
- [ ] Validation queries run after each pipeline execution
- [ ] All checks passed (or failures documented and investigated)
- [ ] Row count tracking table updated in `docs/project_flow.md`

---

### Role 5 -- Documentation and BI

**Owner:** [Unassigned]

**Responsibilities:**
- Keep this wiki and `docs/` up to date as the project evolves
- Update the iterative progress table in `docs/project_flow.md` each week
- Prepare the final presentation: pipeline walkthrough, data lineage, and output screenshots
- Create a Power BI report or Excel summary of `dbo.vw_CustomerSummary` for the final demo
- Ensure the repository README and wiki are clear enough for a new team member to onboard

**Weekly Task Checklist:**
- [ ] `docs/project_flow.md` progress table updated
- [ ] Any wiki pages updated to reflect recent changes
- [ ] Final presentation slides/screenshots prepared (last cycle only)

---

## Iterative Delivery Cycles

See `docs/project_flow.md` for the full cycle breakdown. Below is a summary of role focus per cycle:

| Cycle | Extraction | ETL | Modeling | Validation | Docs |
|---|---|---|---|---|---|
| Cycle 1 (Week 1) | Place sample files | Build first pipeline | Create initial tables | Run first validation | Document setup |
| Cycle 2 (Week 2-3) | Refresh with full data | Automate transforms | Refine schema | Run repeated checks | Record decisions |
| Cycle 3 (Week 4) | Final source refresh | Finalize pipeline | Finalize SQL scripts | Sign-off checklist | Final presentation |

---

## Communication and Coordination

- Use **GitHub Issues** (with the provided issue templates) to track tasks, bugs, and requests
- Link every pull request to an issue using `Closes #<issue-number>` in the PR description
- If you are blocked, open a GitHub issue with the `blocked` label and tag the relevant team member
- Update your role checklist in this page whenever a milestone is completed
