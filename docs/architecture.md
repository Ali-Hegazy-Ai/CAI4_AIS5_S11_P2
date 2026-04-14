# Architecture

This file is kept as a legacy reference. The current project architecture is documented in [wiki/Project-Architecture.md](wiki/Project-Architecture.md).

Use the wiki page for the current, student-project scope and the root docs only when you need historical context.

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
