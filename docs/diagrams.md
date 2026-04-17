# ETL Planning Diagrams

This guide explains how the team works together from raw files to final reporting outputs.

## 1. High-Level Architecture

This diagram shows the full project story, including who performs each major step and how work moves from collection to reporting.

```mermaid
flowchart TD
    S1["Amin and Ali collect CRM and Excel files"] --> S2["Amin and Ali place files in raw storage"]
    S2 --> S3["Ali runs the ETL pipeline in Azure Data Factory"]
    S3 --> S4["Ali standardizes and merges customer records"]
    S4 --> S5["Aseel and Habiba review data quality results"]
    S5 --> S6["Mennatullah and Amin prepare clean data for BI"]
    S6 --> S7["Team reviews insights in Power BI"]

    S3 --- S8["Ali uses secure project secrets during pipeline runs"]
    S3 --- S9["Monitoring tracks each pipeline run"]
    S7 --> S10["Ali, Amin, Mennat Allah, Aseel, and Habiba update documentation"]
```

## 2. Role-Linked Pipeline Flow

This diagram shows who starts each pipeline activity and how ownership passes from ingestion to validated output.

```mermaid
flowchart LR
    P1["Amin triggers CRM file intake"] --> P3["Ali runs transform and publish pipeline"]
    P2["Ali triggers Excel file intake"] --> P3
    P3 --> P4["Aseel and Habiba check run results"]
    P4 --> P5["Mennatullah confirms BI-ready output"]
```

## 3. Mapping Data Flow Steps

This diagram explains Ali's day-to-day transformation work in simple action steps.

```mermaid
flowchart TD
    M1["Ali standardizes email formatting"] --> M2["Ali cleans phone numbers"]
    M2 --> M3["Ali aligns customer name format"]
    M3 --> M4["Ali removes incomplete records"]
    M4 --> M5["Ali applies source priority rules"]
    M5 --> M6["Ali merges duplicate customers"]
    M6 --> M7["Ali assigns stable customer IDs"]
    M7 --> M8["Ali writes cleaned data for validation"]
```

## 4. Pipeline Sequence

This sequence shows team collaboration during one pipeline run from file handoff to monitoring updates.

```mermaid
sequenceDiagram
    participant Amin
    participant Ali
    participant Aseel
    participant Mennatullah
    participant Monitor

    Amin->>Ali: Share latest CRM and Excel files
    Ali->>Ali: Run ETL pipeline and transform data
    Ali->>Aseel: Send cleaned output for quality checks
    Aseel->>Ali: Approve quality or request fixes
    Ali->>Mennatullah: Publish clean dataset for reporting
    Mennatullah->>Monitor: Confirm dashboard refresh
    Monitor->>Amin: Send pipeline status summary
```

## 5. Monitoring and Alerting

This diagram shows how alerts are routed to the right people and who responds first for each issue type.

```mermaid
flowchart LR
    A1["Ali runs scheduled ETL pipeline"] --> A2["Monitor collects logs and run metrics"]
    A2 --> A3["Alert rules evaluate pipeline health"]

    A3 --> A4["Pipeline failure alert"]
    A3 --> A5["Data quality warning"]
    A3 --> A6["Missing file reminder"]

    A4 --> A7["Ali investigates and reruns failed pipeline"]
    A4 --> A8["Aseel and Habiba review failed quality checks"]
    A5 --> A8
    A6 --> A9["Amin and Ali provide missing source files"]
```

## 6. CI/CD and Deployment Flow

This diagram explains how Ali and Mennatullah coordinate releases from development to production with team validation.

```mermaid
flowchart LR
    C1["Ali prepares ETL and pipeline changes in dev branch"] --> C2["Mennatullah reviews reporting impact"]
    C2 --> C3["Ali deploys changes to test environment"]
    C3 --> C4["Aseel and Habiba validate quality in test"]
    C4 --> C5["Ali and Mennatullah approve production release"]
    C5 --> C6["Ali deploys to production"]
    C6 --> C7["Ali, Amin, Mennat Allah, Aseel, and Habiba update docs"]
```

## 7. Project Timeline (4 Weeks)

This timeline shows when each team member is most active and how work transitions across the four weeks.

```mermaid
gantt
    title 4-Week Project Timeline
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section Week 1
    Amin and Ali gather source files :a1, 2026-04-01, 3d
    Ali builds the first ETL flow :a2, after a1, 4d

    section Week 2-3
    Ali automates recurring pipeline runs :b1, after a2, 7d
    Mennatullah and Amin prepare BI-ready output :b2, after b1, 7d
    Aseel and Habiba complete quality checks :b3, after b1, 7d

    section Week 4
    Team validates results and prepares presentation :c1, after b2, 7d
```

Owners by phase:
- Week 1: Amin and Ali lead source preparation and initial pipeline setup.
- Week 2-3: Ali leads automation, while Mennatullah, Amin, Aseel, and Habiba validate output quality and reporting readiness.
- Week 4: The full team finalizes validation, documentation, and presentation.

## 8. Deliverables Checklist

This diagram maps each deliverable to the team members responsible for producing or reviewing it.

```mermaid
flowchart TD
    D1["docs/project_flow.md\nAli, Amin, Mennat Allah, Aseel, Habiba"]
    D2["docs/architecture.md\nAli, Amin, Mennat Allah, Aseel, Habiba"]
    D3["adf/pipelines/pl_transform_merge_publish.json\nAli"]
    D4["sql/scripts/customer_schema.sql\nMennatullah, Amin"]
    D5["data/raw/sample_crm.csv\nAmin, Ali"]
    D6["data/raw/sample_excel.xlsx\nAmin, Ali"]
    D7["data/clean/customers.csv\nAli"]
    D8["presentation/final.pdf\nAli, Amin, Mennat Allah, Aseel, Habiba"]

    Amin["Amin"] -.-> D1
    Amin -.-> D2
    Amin -.-> D4
    Amin -.-> D5
    Amin -.-> D6
    Amin -.-> D8

    Ali["Ali"] -.-> D1
    Ali -.-> D2
    Ali -.-> D3
    Ali -.-> D5
    Ali -.-> D6
    Ali -.-> D7
    Ali -.-> D8

    Mennatullah["Mennatullah"] -.-> D4
    Mennatullah -.-> D8

    Aseel["Aseel"] -.-> D1
    Aseel -.-> D2
    Aseel -.-> D8

    Habiba["Habiba"] -.-> D1
    Habiba -.-> D2
    Habiba -.-> D8
```

## 9. Enterprise Data Platform and AI Architecture

This diagram presents a polished, portfolio-ready left-to-right enterprise architecture for an end-to-end data and AI platform.

```mermaid
flowchart LR
    %% Layer groups in exact required order
    subgraph L1[Ingestion]
      direction TB
      I1["Connectors\nKafka • CDC • Batch Upload"]
      I2["Orchestration\nADF / Airflow Schedules"]
    end

    subgraph L2[Raw Landing]
      direction TB
      R1["Immutable Storage\nBronze Zone / Object Store"]
      R2["Metadata Capture\nSource • Arrival Time • Schema"]
    end

    subgraph L3[Validation and Quarantine]
      direction TB
      V1["Data Quality Rules\nSchema • Nulls • Duplicates"]
      V2["Quarantine Queue\nRejected Records + Issue Codes"]
    end

    subgraph L4[Transformation and Feature Engineering]
      direction TB
      T1["Standardization\nType Casting • Business Logic"]
      T2["Feature Pipelines\nAggregates • Time Windows"]
    end

    subgraph L5[Warehouse Modeling]
      direction TB
      W1["Dimensional Models\nStar/Snowflake Mart Layers"]
      W2["Performance Layer\nPartitioning • Materialized Views"]
    end

    subgraph L6[Governance and Lineage]
      direction TB
      G1["Catalog & Policies\nPII Tags • RBAC • Retention"]
      G2["Lineage Graph\nColumn-Level Traceability"]
    end

    subgraph L7[BI Layer]
      direction TB
      B1["Semantic Models\nMetrics Store • KPIs"]
      B2["Dashboards\nExecutive & Operational Analytics"]
    end

    subgraph L8[ML and NLP]
      direction TB
      M1["Model Training\nForecasting • Classification"]
      M2["NLP Pipelines\nEntity Extraction • Sentiment"]
    end

    subgraph L9[LLM Layer]
      direction TB
      LL1["Prompt & Context Engine\nRAG • Guardrails • Caching"]
      LL2["Model Ops\nRouting • Evaluation • Cost Controls"]
    end

    subgraph L10[Serving APIs]
      direction TB
      S1["Data Products API\nREST / GraphQL / gRPC"]
      S2["Inference Endpoints\nReal-Time Scoring + GenAI"]
    end

    subgraph L11[Feedback Loop and Retraining]
      direction TB
      F1["Observability\nDrift • Latency • Quality Signals"]
      F2["Continuous Improvement\nHuman Feedback -> Retraining"]
    end

    %% Primary left-to-right flow
    L1 --> L2 --> L3 --> L4 --> L5 --> L6 --> L7 --> L8 --> L9 --> L10 --> L11

    %% Operational feedback paths
    L11 -. Model/Prompt Updates .-> L8
    L11 -. Policy & Data Quality Insights .-> L6
    L11 -. Product Usage Signals .-> L10

    %% Enterprise styling
    style L1 fill:#eaf4ff,stroke:#7aa7d9,stroke-width:1.5px,rx:8,ry:8
    style L2 fill:#eef7ff,stroke:#7aa7d9,stroke-width:1.5px,rx:8,ry:8
    style L3 fill:#f2f8ff,stroke:#7aa7d9,stroke-width:1.5px,rx:8,ry:8
    style L4 fill:#eefaf4,stroke:#69b28f,stroke-width:1.5px,rx:8,ry:8
    style L5 fill:#ecf9f0,stroke:#69b28f,stroke-width:1.5px,rx:8,ry:8
    style L6 fill:#fff7ea,stroke:#c49a57,stroke-width:1.5px,rx:8,ry:8
    style L7 fill:#fff3ef,stroke:#c98070,stroke-width:1.5px,rx:8,ry:8
    style L8 fill:#f5f1ff,stroke:#8d78c6,stroke-width:1.5px,rx:8,ry:8
    style L9 fill:#f0edff,stroke:#8d78c6,stroke-width:1.5px,rx:8,ry:8
    style L10 fill:#eef4ff,stroke:#6787c8,stroke-width:1.5px,rx:8,ry:8
    style L11 fill:#f9fbff,stroke:#5f7fb2,stroke-width:1.5px,rx:8,ry:8
```
