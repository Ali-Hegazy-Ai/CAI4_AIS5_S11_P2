# Home - Customer Data ETL Wiki

Welcome to the official wiki for the **Customer Data ETL** project.

Whether you are a new team member joining mid-project or a student trying to understand data engineering for the first time, this wiki will walk you through everything -- from what ETL is, to how to run the pipeline yourself.

---

## What Is This Project?

**Customer Data ETL** is a data engineering pipeline that collects raw customer records from two separate business systems -- a **CRM export (CSV)** and an **Excel spreadsheet** -- and automatically:

1. **Extracts** the data from those source files
2. **Transforms** it (cleans, standardizes, deduplicates, and merges)
3. **Loads** it into a clean, analysis-ready SQL Server table

The result is a single, reliable source of customer data that a BI analyst or reporting tool can query directly.

---

## Why Do ETL Pipelines Matter?

In the real world, customer data is rarely clean or centralized. It lives in many places -- CRM systems, spreadsheets, databases -- and each source uses different formats, column names, and quality standards.

Without an ETL pipeline:
- Analysts waste hours manually cleaning spreadsheets
- Reports show inconsistent numbers because each team queries a different source
- Duplicate or missing records cause business decisions to be based on wrong data

An ETL pipeline solves this by **automating the journey from raw, messy data to clean, trusted data**, reliably and repeatably.

---

## What Problem Does This Project Solve?

This project integrates two customer data sources that use different formats and naming conventions. Left unmerged, these sources produce duplicate entries, conflicting values, and gaps in reporting. The pipeline:

- Merges both sources into one canonical customer record per `CustomerID`
- Standardises column names, date formats, phone formats, and text casing
- Removes duplicate and null-key records
- Loads the result into SQL Server for downstream analytics

---

## High-Level Architecture

```
+------------------------------------------------------------------+
|                          DATA SOURCES                            |
|                                                                  |
|   +------------------+          +-----------------------+        |
|   |  CRM Export      |          |  Excel Spreadsheet    |        |
|   |  (CSV file)      |          |  (.xlsx file)         |        |
|   +--------+---------+          +----------+------------+        |
|            |                               |                     |
|            +---------------+---------------+                     |
+----------------------------|-----------------------------------------+
                             |
                             v
+------------------------------------------------------------------+
|              AZURE DATA FACTORY (Orchestration)                  |
|                                                                  |
|   Pipeline: pl_customer_etl                                      |
|                                                                  |
|   [Copy CRM] --> [Copy Excel] --> [Data Flow: Transform]         |
|                                         |                        |
|             +---------------------------+                        |
|             |  Data Flow Steps:                                  |
|             |  1. Select (standardize column names)             |
|             |  2. Derived Column (fix types, formats)           |
|             |  3. Filter (remove null-key rows)                 |
|             |  4. Deduplicate (keep one record per CustomerID)  |
|             |  5. Union (merge CRM + Excel streams)             |
|             +---------------------------+                        |
|                                         |                        |
|   --> [Copy to Clean CSV] --> [Load to SQL Table]                |
+------------------------------------------------------------------+
                             |
              +--------------+--------------+
              v                             v
+---------------------------+   +---------------------------+
|   data/clean/             |   |  SQL Server               |
|   (cleaned CSV output)    |   |  dbo.Customers            |
+---------------------------+   +---------------------------+
```

---

## Tech Stack

| Tool | Role |
|---|---|
| **Azure Data Factory** | Orchestrates the pipeline and runs Data Flows |
| **ADF Data Flow** | Visual transformation engine (merge, clean, deduplicate) |
| **Azure Blob Storage** | Hosts raw source files and clean output files |
| **SQL Server / Azure SQL** | Target database for clean customer records |
| **Excel / CSV** | Source data formats |
| **Git / GitHub** | Version control for all code, scripts, and ADF assets |

---

## Repository Structure

```
customer-data-etl/
+-- data/
|   +-- raw/              <- Drop source files here (never edit them)
|   +-- clean/            <- Pipeline writes clean output here
+-- sql/
|   +-- scripts/          <- SQL scripts: tables, views, procedures
+-- adf/
|   +-- pipelines/        <- ADF pipeline JSON exports
|   +-- datasets/         <- ADF dataset JSON exports
|   +-- linked_services/  <- ADF connection JSON exports
+-- docs/
|   +-- README.md         <- Project guide and team rules
|   +-- project_flow.md   <- Iterative delivery phases
+-- wiki/                 <- This wiki (all pages live here)
```

---

## Wiki Navigation

| Page | What you will learn |
|---|---|
| [Home](Home) | Project overview, architecture, and this navigation guide |
| [Project Architecture](Project-Architecture) | Deep dive into each layer of the pipeline |
| [Setup Guide](Setup-Guide) | Clone the repo and configure your environment step by step |
| [ETL Pipeline](ETL-Pipeline) | How the ADF pipeline works end to end |
| [Data Sources](Data-Sources) | CRM and Excel source schemas, formats, and quality issues |
| [SQL Schema](SQL-Schema) | Warehouse table design, views, and stored procedures |
| [Data Validation](Data-Validation) | Quality checks and validation queries to run after each load |
| [Team Roles](Team-Roles) | Who owns what -- role assignments and weekly responsibilities |
| [Contributing](Contributing) | How to branch, commit, open pull requests, and use issue templates |
| [Glossary](Glossary) | Plain-English definitions of every technical term used here |

---

## Team Members

| Name | Role | GitHub |
|---|---|---|
| Ali | Project Lead / Architecture | @Ali-Hegazy-Ai |
| Amin | ETL / ADF Pipeline | TBD |
| Mennat Allah | Data Modeling / SQL | TBD |
| Aseel | Data Validation / Testing | TBD |
| Habiba | Documentation / BI | TBD |

---

> **New to ETL?** Start with the [Glossary](Glossary) to learn the key terms, then read [Project Architecture](Project-Architecture) to see the big picture before diving into technical details.
