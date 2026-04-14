# Raw Layer Governance Rules

## Purpose and Scope
- The raw layer stores original ingested data exactly as received from source systems.
- The raw layer exists to guarantee full traceability and deterministic reprocessing.
- These rules are strict and mandatory for all pipelines writing to raw storage.

## Core Principles

### No Transformation
- Raw payload content must not be transformed, cleaned, standardized, deduplicated, or enriched.
- Field values, file structure, and record ordering must remain as provided by the source.
- Allowed operations are limited to storage-safe operations that do not alter business content:
  - Chunking
  - Lossless compression
  - Checksum generation
  - Metadata attachment

### Immutable Data
- Raw data is write-once and read-many.
- Existing raw files must never be edited in place.
- Corrections, late arrivals, and re-ingestions must be written as new files and/or new versions.

### Full Traceability
- Every raw file must be traceable to:
  - Source system
  - Ingestion execution time
  - Ingestion batch
  - Original file location or endpoint
- Lineage must allow a complete audit trail from raw storage back to origin.

## File Organization

### Folder Hierarchy by Source
Use a source-first layout to isolate provenance and simplify governance checks.

Example hierarchy:
- raw/source_name/dataset_name/ingestion_date=YYYY-MM-DD/batch_id=.../

Rules:
- One source per source folder.
- One dataset or entity per dataset folder.
- No mixing unrelated sources or datasets in the same leaf folder.

### Partitioning Strategy (Date and Source)
- Required partition keys:
  - source_name
  - ingestion_date (UTC)
- Recommended additional partition keys for high volume:
  - ingestion_hour (UTC)
  - batch_id
- Partition values must be derived from ingestion metadata, not transformed business fields.

## Naming Conventions

### File Naming Standard
Use a deterministic and parseable file name format:
- sourceName__datasetName__ingestTsUTC__batchId__part-XXX-of-YYY__vN.ext

Naming rules:
- sourceName and datasetName must use canonical registry names.
- ingestTsUTC must be in UTC and include full timestamp.
- part-XXX-of-YYY is mandatory for chunked files; use part-001-of-001 for single-file loads.
- vN is the immutable raw file version for that logical object.

### Batch Identifiers
- Every ingestion execution must have a unique batch_id.
- batch_id must be immutable and never reused.
- Recommended format:
  - BYYYYMMDDHHMMSS_sourceName_seqNNN
- Retries must use a new batch_id and reference the prior failed batch in metadata.

## Metadata Requirements
The following fields are mandatory for every raw file and every chunk.

| Field | Requirement |
| --- | --- |
| ingestion_timestamp | UTC timestamp of when the platform committed the object to raw storage |
| source_name | Canonical source system name from governed source registry |
| batch_id | Unique ingestion execution identifier |
| file_origin | Original file path, API endpoint, table, or object URI from the source |

Additional governance requirements:
- Metadata must be persisted with the object (sidecar, manifest, or object tags) and queryable.
- Missing mandatory metadata means ingestion is invalid and must fail fast.

## Data Integrity Rules

### No Overwriting
- Overwrite operations are prohibited in raw storage.
- If an object with the same logical identity already exists, write a new version instead of replacing content.
- Delete and recreate behavior is prohibited for normal ingestion and reprocessing workflows.

### Versioning Strategy
- Use append-only versioning for every logical raw object.
- Version rules:
  - Initial load is v1.
  - Re-ingestion or correction creates v2, v3, and so on.
  - Prior versions remain accessible for audit and replay.
- Maintain checksum records (recommended: SHA-256) per version to verify content integrity.

## Handling Large Files

### Chunking
- Large files must be chunked when they exceed platform-defined limits.
- Chunking rules:
  - Use deterministic chunk sizes per source profile.
  - Preserve chunk order using part numbers in file names.
  - All chunks from one source object must share the same batch_id and origin reference.
  - Reassembly instructions must be derivable from metadata and naming.

### Compression (If Applicable)
- Compression is optional but recommended for very large raw payloads.
- Only lossless compression is allowed.
- Compression must not remove records, alter values, or change semantic content.
- Compression algorithm and settings must be recorded in metadata.

## Anti-Patterns (Important)
The following must NEVER happen in the raw layer:

- Transforming, cleansing, deduplicating, filtering, or enriching raw records.
- Overwriting or editing existing raw files in place.
- Deleting prior versions required for traceability or replay.
- Reusing batch_id across different ingestion executions.
- Storing files without mandatory metadata fields.
- Mixing multiple sources in a single source partition.
- Writing files to partitions derived from transformed business dates instead of ingestion metadata.
- Manually uploading or editing raw files outside controlled ingestion pipelines.
- Marking partial or failed loads as complete.
- Using lossy compression or proprietary formats that block reliable reprocessing.
