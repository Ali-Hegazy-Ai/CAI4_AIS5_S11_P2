## Extraction Architecture Overview
This architecture is designed for 150MB-500MB files and multi-million-record ingestion across CSV, API, and JSON sources on Azure. The extraction layer emphasizes horizontal scale, idempotent execution, and fast recovery from partial failures.

- Orchestration: Azure Data Factory (ADF) metadata-driven pipelines.
- Storage target for raw ingestion: Azure Data Lake Storage Gen2 (ADLS Gen2) immutable raw zone.
- Distributed compute for heavy parsing/chunking: Azure Databricks (Spark).
- Control state: ingestion control tables (Azure SQL or Delta) for batch state, checkpoints, and watermarks.
- Monitoring: Azure Monitor + Log Analytics with batch-level correlation IDs.

### CSV (Kaggle)
### Ingestion Strategy Table
| Source Type | Method | Tool | Mode | Frequency | Scalability Approach |
| --- | --- | --- | --- | --- | --- |
| CSV (Kaggle) | Bulk file pull to landing, then distributed chunked load to raw | ADF Copy Activity + ADLS Gen2 + Databricks (Spark) | Chunked micro-batch | Scheduled (hourly/daily) + on-demand backfill | File-level and partition-level parallelism, autoscaling Spark, checkpointed chunk commits |

#### Chunked processing strategy
- Land each CSV file unchanged in raw landing first, then process in distributed chunks.
- Configure Spark input split sizing (for example 128MB-256MB) so executors process partitions without loading the entire file into memory.
- Use chunk checkpoints (`chunk_id`, `file_name`, `row_count`, `status`) in a control table for restartability.
- Commit output per chunk so failures reprocess only failed chunks, not the entire file.

#### Parallel ingestion considerations
- Use ADF `ForEach` with controlled concurrency for file-level parallel loads.
- For very large single files, rely on Spark partition parallelism (`repartition`) for intra-file parallel processing.
- Tune concurrency limits using ADLS throughput and Spark executor core availability.
- Isolate high-volume ingestion pipelines from lower-priority jobs through separate integration runtimes or compute pools.

#### File partitioning strategy
- Primary partition path by source and ingestion date: `raw/source=csv_kaggle/dataset=<dataset_name>/ingestion_date=YYYY-MM-DD/batch_id=<uuid>/`.
- If a reliable business date exists in data, add secondary partition (`event_date=YYYY-MM-DD`).
- For skewed distributions, add hash bucket partitioning (`bucket_id=00..NN`) to improve downstream parallel reads.

#### Handling large files efficiently
- Keep a compressed copy (`.csv.gz`) in landing for network/storage efficiency.
- Parse and write raw curated files in columnar format (`parquet` + `snappy`) for high-volume scan efficiency, while preserving original file copy.
- Enforce explicit schemas where known to avoid repeated expensive type inference.
- Validate integrity with source byte size, row counts, and checksum reconciliation.

### API
### Ingestion Strategy Table
| Source Type | Method | Tool | Mode | Frequency | Scalability Approach |
| --- | --- | --- | --- | --- | --- |
| API | REST extraction with pagination + watermark-based incremental pulls | ADF REST Connector + Azure Functions (optional throttle/retry wrapper) + ADLS Gen2 | Incremental micro-batch | Every 5-15 min for high-change APIs; hourly/daily otherwise | Endpoint fan-out, paginated parallel pulls within safe limits, centralized watermark checkpoints |

#### Pagination handling
- Support cursor, token, offset/limit, and next-link patterns through parameterized pipeline activities.
- Persist last successful page token and extraction boundary per endpoint in control metadata.
- Use continuation loops until no next token or record count threshold is reached.
- For APIs with unstable page ordering, paginate on immutable keys/time windows to avoid duplicates/misses.

#### Rate limiting strategy
- Respect provider quotas from headers (`X-RateLimit-*`, `Retry-After`) and published limits.
- Apply endpoint-specific concurrency caps (for example max parallel calls per endpoint).
- Use a shared throttling mechanism (Function/app-level token bucket) when many pipelines hit the same API.
- Schedule heavy endpoints in staggered windows to avoid synchronized spikes.

#### Retry + backoff logic
- Classify failures as transient (timeouts, HTTP 429, HTTP 5xx -> retry) and non-transient (HTTP 4xx validation/auth issues -> fail fast and alert).
- Use exponential backoff with jitter (for example base 2s, factor 2, capped at 60s).
- Enforce max retry attempts per page/request and dead-letter unrecoverable payload references.
- Make request processing idempotent via request signature (`endpoint`, `window_start`, `window_end`, `page_token`).

#### Incremental extraction (if possible)
- Prefer API filters based on `updated_at`, `modified_since`, or incremental IDs.
- Store and advance watermark only after successful batch commit to raw storage.
- Add overlap windows (for example re-read last 5-15 minutes) to protect against late updates; deduplicate downstream by natural key + latest timestamp.
- Run periodic reconciliation snapshots to detect silent misses.

### JSON (local)
### Ingestion Strategy Table
| Source Type | Method | Tool | Mode | Frequency | Scalability Approach |
| --- | --- | --- | --- | --- | --- |
| JSON (local) | Batched local-to-cloud transfer, then distributed parse and load | ADF with Self-hosted Integration Runtime + ADLS Gen2 + Databricks (for nested normalization) | Batch and micro-batch | Every 15-60 min + nightly catch-up | Batch manifests, file-level parallel transfer, distributed flattening for nested payloads |

#### Batch ingestion
- Collect local JSON files into batch manifests (`batch_id`, file list, expected size/count).
- Transfer batches through Self-hosted IR to ADLS raw landing in parallel lanes.
- Process landed files in Spark using multi-file input and controlled partition counts.
- Track file-level status to allow partial batch replay without duplicating successful files.

#### Schema inference challenges
- JSON often has nested objects/arrays, optional fields, and type drift across files.
- Pure auto-inference can cause schema instability and expensive reprocessing.
- Use schema evolution policy with guarded promotion rules (for example int -> long allowed, string -> struct blocked unless approved).
- Keep unknown/new fields in a rescue column (or raw JSON payload column) for later controlled evolution.
- Version schema definitions and tie each ingestion batch to schema version metadata.

## Raw Layer Loading Strategy

### Folder structure
- Recommended ADLS raw zone path pattern: `raw/source=<source_type>/system=<source_system>/entity=<entity_name>/ingestion_date=YYYY-MM-DD/ingestion_hour=HH/batch_id=<uuid>/`.
- Optional technical subfolders include `landing/` for immutable originals, `processed/` for normalized raw-curated files, and `quarantine/` for malformed records/files.

### Naming conventions
- File name pattern: `<source>_<entity>_<extract_start_utc>_<extract_end_utc>_<batch_id>_<part_number>.<ext>`.
- Standards: lowercase snake_case for folders and entities, UTC timestamps in ISO-like sortable form (for example `2026-04-14T23-00-00Z`), and immutable `batch_id` (UUID) per ingestion run.

### Partitioning (by date/source)
- Mandatory partitions: `source`, `ingestion_date`.
- Recommended additional partitions for high-volume entities: `entity`, `ingestion_hour`.
- If source provides event time and downstream workloads are event-time driven, include `event_date` partition in processed raw outputs.

### Metadata columns
Add these metadata columns to every extracted record (or sidecar manifest when record-level enrichment is not possible at landing):
- `ingestion_timestamp`: UTC timestamp when record/file entered cloud raw layer.
- `source`: Source identifier (for example `csv_kaggle`, `api_salesforce`, `json_local`).
- `batch_id`: Unique ingestion batch identifier for traceability and replay control.

## Failure Handling Strategy

### Partial ingestion
- Track state at granular level: CSV at chunk-level, API at page/window-level, JSON at file-level.
- Use idempotent write keys (`batch_id` + technical unit id) so reruns do not duplicate completed units.
- Commit checkpoint only after durable write to ADLS succeeds.
- Route malformed records/files to `quarantine/` with reason codes; continue processing valid units when safe.

### Retry policies
- Configure activity-level retries in ADF for transient connector/network failures.
- Apply exponential backoff with jitter in API and custom ingestion components.
- Separate retry budgets by error class: short retry loop for 429/503 and longer retry loop for temporary network/storage issues.
- After retry exhaustion, mark batch as `FAILED_RETRY_EXHAUSTED`, alert operations, and preserve restart checkpoints.

### Logging approach
- Emit structured logs with consistent fields: `batch_id`, `source`, `entity`, `file_or_page_id`, `status`, `attempt`, `duration_ms`, `rows_read`, `rows_written`, `error_code`.
- Centralize logs/metrics in Azure Monitor and Log Analytics.
- Build operational dashboards for throughput, failure rates, lag, and top error categories.
- Trigger alerts for SLA breaches, repeated retries, and zero-record anomalous loads.

## Performance Considerations

### Memory constraints
- Never load full 150MB-500MB files into single-process memory.
- Use streaming/chunked readers and Spark partitioned processing.
- Size executors for stable memory headroom; avoid oversized partitions that cause spill and OOM.
- Prefer column pruning and predicate pushdown in downstream reads of columnar raw-curated outputs.

### I/O bottlenecks
- Parallelize read/write paths while respecting ADLS account throughput limits.
- Reduce small-file explosion by compaction targets (for example 128MB-512MB output files).
- Use compression (`snappy` for parquet, gzip for raw text transport) to balance CPU vs I/O.
- Keep ingestion compute in same Azure region as ADLS to reduce network latency and egress overhead.

### Parallelism strategy
- Apply layered parallelism across source-level (independent source pipelines), unit-level (files/pages/chunks), and partition-level (distributed Spark execution) workloads.
- Use autoscaling compute with concurrency guardrails to prevent shared resource contention.
- Tune ADF pipeline concurrency, Spark shuffle partitions, and API call parallelism together as one capacity model.
- Validate tuning with load tests that reflect peak file sizes and record counts, not average-day volumes.