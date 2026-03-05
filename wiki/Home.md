# Customer Data ETL — Wiki Home

Welcome to the wiki for the **Customer Data ETL** project.

This is a college group project (DEPI Final Project) that integrates customer data from CRM and Excel sources, transforms it using Azure Data Factory, and loads clean data ready for Data Warehouse analysis.

---

## Team Members

| Name | GitHub Handle |
|---|---|
| Ali | @Ali-Hegazy-Ai |
| Amin | TBD |
| Mennat Allah | TBD |
| Aseel | TBD |
| Habiba | TBD |

---

## Wiki Pages

| Page | Description |
|---|---|
| [Home](Home) | This page — project overview and navigation |
| [Setup Guide](Setup-Guide) | How to clone the repo and set up your environment |
| [ETL Pipeline](ETL-Pipeline) | Azure Data Factory pipeline architecture and data flow |
| [Data Sources](Data-Sources) | CRM and Excel source file schemas and formats |
| [SQL Schema](SQL-Schema) | Warehouse table design and SQL scripts guide |
| [Data Validation](Data-Validation) | Quality checks, test queries, and validation rules |
| [Team Roles](Team-Roles) | Role ownership, responsibilities, and weekly tasks |

---

## Project Goal

Merge customer data from CRM and Excel, clean and transform it using ADF Data Flows, and produce a clean output ready for Data Warehouse / BI reporting.

## Tech Stack

- **Azure Data Factory** — orchestration and data flows
- **SQL Server** — target schema and transformation scripts
- **Excel / CSV** — source data files

## Repository Structure

```
customer-data-etl/
├── data/
│   ├── raw/              # Original source files (CRM export, Excel)
│   └── clean/            # Final cleaned files ready for analysis
├── sql/
│   └── scripts/          # SQL scripts for tables, views, and load steps
├── adf/
│   ├── pipelines/        # ADF pipeline JSON definitions
│   ├── datasets/         # ADF dataset JSON definitions
│   └── linked_services/  # ADF linked service connection definitions
├── docs/
│   ├── README.md         # Project guide
│   └── project_flow.md   # Team phases and iterative workflow
└── wiki/                 # Wiki pages (this directory)
```

## Quick Links

- [Project Guide](../docs/README.md)
- [Project Flow & Phases](../docs/project_flow.md)
- [Contributing Guide](../CONTRIBUTING.md)
