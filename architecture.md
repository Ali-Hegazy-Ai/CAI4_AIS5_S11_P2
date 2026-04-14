## High-Level Architecture

This ETL platform is designed as a cloud-first, metadata-driven pipeline on Azure, optimized for heterogeneous inputs (CSV, API, JSON) and high-volume processing (hundreds of MB per file, millions of rows per load window).

The architecture is organized into six layers:

1. Sources
- Batch file producers (CSV exports from CRM/ERP systems)
- REST APIs with pagination/rate limits
- Semi-structured JSON drops (system logs, partner feeds, app telemetry snapshots)

2. Ingestion
- Azure Data Factory (ADF) orchestrates source-specific extract pipelines
- Parameterized datasets and linked services reduce duplicated pipeline logic
- Watermark-based and API token-based incremental ingestion patterns are applied

3. Raw Storage
- Azure Blob Storage is the immutable landing zone
- Source payloads are stored in original format for replay/audit
- Raw data is partitioned by source, entity, and ingest timestamp for scalable reads

4. Transformation
- ADF Mapping Data Flows perform schema normalization, type enforcement, deduplication, and quality checks
- Optional Python components handle advanced JSON flattening, custom parsing, and exceptional business rules

5. Serving Layer
- Azure SQL hosts conformed, query-optimized tables (dimensions/facts or standardized entity tables)
- Loads use staging + MERGE (upsert) semantics for idempotent refreshes

6. Analytics
- BI tools (Power BI or equivalent) consume curated SQL views and semantic-ready tables
- Data consumers query stable, business-friendly schemas rather than raw operational structures

## Architecture Diagram (Mermaid)

```mermaid
flowchart TB
    subgraph SRC[Sources]
        CSV[CSV Files\nCRM / ERP Exports]
        API[REST APIs\nPaginated / Rate-Limited]
        JSON[JSON Drops\nPartner / App Feeds]
    end

    subgraph ING[Ingestion Layer - Azure Data Factory]
        ORCH[Metadata-Driven Orchestration\nPipeline + Trigger + Retry Policy]
        COPY[Copy Activities\nParallelized by Entity]
        INC[Incremental Controls\nWatermark / API Cursor / Last-Modified]
    end

    subgraph STOR[Storage Layer - Azure Blob Storage]
        RAW[Raw Zone (Immutable)\n/source/entity/ingest_date/run_id]
        STG[Staging Zone\nNormalized Parquet/CSV + QC Flags]
        CUR[Curated Zone\nConsumption-Ready Exports]
    end

    subgraph XFORM[Transformation Layer]
        DF[ADF Mapping Data Flows\nSchema Align + Type Cast + Dedup]
        PY[Optional Python\nComplex JSON Flatten + Rule Engines]
    end

    subgraph SERV[Serving Layer - Azure SQL]
        STGTBL[Staging Tables\nBulk Load Target]
        MERGE[MERGE Procedures\nIdempotent Upserts]
        MART[Curated Tables + Views\nAnalytics Contract]
    end

    subgraph ANA[Analytics]
        PBI[Power BI / SQL Clients\nDashboards + Ad-hoc Analysis]
    end

    CSV --> ORCH
    API --> ORCH
    JSON --> ORCH

    ORCH --> COPY
    COPY --> INC
    INC --> RAW

    RAW --> DF
    RAW --> PY
    PY --> STG
    DF --> STG

    STG --> CUR
    STG --> STGTBL

    STGTBL --> MERGE
    MERGE --> MART

    MART --> PBI
    CUR --> PBI
```

### Sources Layer

The Sources layer handles three ingestion classes with explicit contracts:

1. File-based CSV sources
- Expected with source-side schema drift risk (new columns, reordered columns, malformed delimiters)
- Contract controls: header validation, file checksum logging, row count capture, schema fingerprinting

2. API sources
- Often paginated and quota-constrained
- Contract controls: cursor/token persistence, retry with exponential backoff, HTTP status stratification (4xx vs 5xx), throttling-aware call cadence

3. JSON sources
- Semi-structured and nested, with optional arrays and sparse fields
- Contract controls: schema version detection, nullability defaults, controlled flattening depth, dead-letter route for invalid payloads

Design note: each source must provide a stable business key (or key composition logic) before data can enter curated domains.

### Ingestion Layer

Ingestion is orchestrated with Azure Data Factory using a metadata-driven pattern:

1. Control metadata
- Stored in Azure SQL control tables (example: `etl.SourceConfig`, `etl.LoadWindow`, `etl.RunAudit`)
- Defines source type, endpoint/path, authentication reference, incremental key, target entity, and schedule profile

2. Pipeline topology
- One master pipeline enumerates active entities from metadata
- Child pipelines execute per entity with parameterized datasets and linked services
- Parallelism is controlled via batch count to maximize throughput without overloading source systems

3. Reliability controls
- Activity-level retries with bounded backoff
- Poison message handling (invalid source files/API pages routed to quarantine)
- Run-state persistence for restartability from the last successful checkpoint

4. Security
- Credentials in Azure Key Vault (referenced by ADF linked services)
- Private endpoints/VNet integration for storage and SQL where required

### Storage Layer (Raw / Staging / Curated)

Storage is zone-based inside Azure Blob Storage to isolate concerns and support replayability.

1. Raw zone
- Immutable source copy; no updates in place
- Path convention example:
  - `raw/<source_system>/<entity>/ingest_date=YYYY-MM-DD/hour=HH/run_id=<guid>/payload.*`
- Retains original CSV/JSON/API response payloads for audit/reprocessing

2. Staging zone
- Intermediate normalized data after basic structural alignment
- Preferred format: Parquet for large tabular datasets (column pruning + compression)
- Includes quality attributes: `dq_status`, `dq_error_code`, `ingestion_run_id`, `record_hash`

3. Curated zone
- Business-ready exports for downstream consumption and interoperability
- Includes denormalized snapshots and extracts required by non-SQL consumers

Retention model:
- Raw: longer retention for replay/compliance
- Staging: shorter retention for operational troubleshooting
- Curated: retention based on analytics SLAs and backfill requirements

### Transformation Layer

Transformation combines ADF Data Flows with optional Python for non-trivial logic:

1. Standardized transformations (ADF)
- Column mapping and canonical naming
- Type casting with strict failure capture
- Null handling/default imputation policies
- Deduplication by business key + freshness rule (`last_update_ts` precedence)

2. Advanced transformations (Python, optional)
- Deep JSON flattening for dynamic nested structures
- Complex entity resolution when keys are inconsistent across systems
- Specialized parsing (locale-specific dates, free-text normalization, rule-based classification)

3. Data quality gates
- Mandatory key presence
- Domain/range checks for critical numeric/date fields
- Referential checks before serving-layer merge
- Rule violations split into reject streams with full lineage identifiers

4. Idempotency
- Transform outputs are keyed by `run_id` and entity partition
- Reprocessing the same run does not duplicate serving-layer records

### Serving Layer

Azure SQL is used as the serving contract for analytics and downstream applications.

1. Load strategy
- Bulk load transformed records into SQL staging tables
- Stored-procedure MERGE into curated tables using business keys
- Separate insert/update counters captured per run for observability

2. Physical design for scale
- Partition large fact-like tables by date (for prune-friendly queries and maintenance)
- Use clustered columnstore indexes for large analytical tables where scan workloads dominate
- Use rowstore indexes for selective lookup dimensions and merge predicates

3. Data contract
- Consumer-facing SQL views shield BI tools from schema evolution
- Backward-compatible view strategy minimizes dashboard breakage

4. Operational controls
- Run audit tables capture start/end time, row counts, reject counts, and failure reason
- Late-arriving data policy defines whether to restate historical partitions or append corrections

## Technology Stack Justification

Azure Data Factory
- Chosen for enterprise orchestration, native connectors, managed scaling, and visual pipeline governance
- Supports mixed source modalities (files, APIs, databases) under a single control plane
- Enables metadata-driven parameterization and operational monitoring without custom scheduler overhead

Azure Blob Storage
- Cost-effective durable landing zone for raw + intermediate + curated files
- Supports large object storage and high-throughput parallel read/write patterns
- Natural fit for immutable raw retention and replay-based recovery

Azure SQL
- Strong serving layer for structured, governed, query-optimized datasets
- Supports transactional merge logic, indexing strategies, and stable SQL contracts for BI
- Good balance between operational simplicity and analytical capability for this workload profile

Optional Python Components
- Adds flexibility for transformations that are awkward in purely visual flows (deeply nested JSON, custom parsers, sophisticated rule engines)
- Keeps advanced logic modular and testable while ADF remains the orchestrator
- Used selectively to avoid over-customization of the core ETL path

Why this combination works together
- ADF handles orchestration and connectivity
- Blob provides scalable storage tiers and replayability
- SQL provides governed serving semantics and BI-friendly access
- Python fills the capability gap only where needed

## Data Flow Description

1. Source discovery and load window resolution
- Master pipeline reads active source/entity definitions from control metadata
- Determines incremental boundaries (last watermark, API cursor, file date window)

2. Parallel extraction
- For each entity, ADF pulls source data in parallel (bounded by configured concurrency)
- File and API payloads are persisted exactly as received into Raw zone with run metadata

3. Structural normalization
- Raw payloads are parsed into Staging schema
- Column names, datatypes, and structural shape are standardized

4. Data quality and enrichment
- Mandatory fields validated; invalid records routed to reject paths with reason codes
- Business enrichments applied (derived dimensions, standardized codes, calculated attributes)

5. Curated dataset preparation
- Cleaned data is written to Curated zone exports for interchange and traceability
- Same cleaned batch is bulk-loaded into Azure SQL staging tables

6. Serving merge and publish
- SQL MERGE procedures upsert into curated serving tables
- Consumer views are refreshed/validated for downstream analytics

7. Monitoring and audit closure
- Pipeline updates run audit with throughput metrics, success/failure state, and reject statistics
- Alerting hooks can notify operators on threshold breaches or failed entities

## Scalability Considerations

Large file handling
- Use chunked/partition-aware processing in ADF where possible to avoid single-thread bottlenecks
- Convert staging outputs to Parquet for compression and faster downstream scans
- Avoid repeated full-file reprocessing by using incremental windows and watermark filters

Horizontal scaling
- Drive ingestion fan-out through metadata so new entities scale by configuration, not code duplication
- Tune ADF integration runtime and activity concurrency to exploit parallel copy/transform execution
- Partition storage paths and SQL tables to keep load/query windows bounded

Future streaming support
- Add an event-driven ingress path (for example: Event Hubs -> micro-batch landing in Raw)
- Reuse the same zone model (Raw -> Staging -> Curated) for batch and near-real-time data
- Preserve serving contract in Azure SQL while introducing low-latency append/merge cadence

## Tradeoffs and Design Decisions

1. ADF-first orchestration vs custom code orchestration
- Decision: ADF-first
- Benefit: managed operations, faster onboarding, lower scheduler maintenance
- Tradeoff: very complex conditional logic can be less ergonomic than code-centric orchestrators

2. Blob zone architecture vs direct-to-SQL ingestion
- Decision: keep Raw and Staging in Blob before SQL serving
- Benefit: replayability, lineage, decoupling extraction from serving failures
- Tradeoff: additional storage and one extra hop increases end-to-end latency slightly

3. SQL serving layer vs lake-only consumption
- Decision: Azure SQL as primary serving contract
- Benefit: stable SQL interface, easier BI adoption, clear governance boundary
- Tradeoff: requires careful index/partition maintenance as volumes increase

4. Visual transforms vs Python extensibility
- Decision: use visual transforms by default, Python only for high-complexity cases
- Benefit: maintainability for most team members while preserving technical flexibility
- Tradeoff: dual transformation paradigms require discipline in testing and ownership

5. Incremental loads vs full refresh
- Decision: incremental by default; full backfill only on demand
- Benefit: lower runtime and compute cost on large datasets
- Tradeoff: requires robust watermark/state management and late-arrival handling rules

6. Strict quality gates vs permissive ingestion
- Decision: strict gates on key business fields; quarantine invalid records
- Benefit: protects downstream analytics trust
- Tradeoff: may delay availability when source quality degrades, requiring clear remediation workflows
