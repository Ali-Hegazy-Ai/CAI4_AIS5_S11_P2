# Sources Catalog

## Source Inventory Summary

| Source | Type | Estimated Size | Estimated Volume (rows) | Refresh Pattern | Ingestion Complexity |
|---|---|---:|---:|---|---|
| Kaggle - Customer 360 Bundle | CSV | 480 MB | 25.6M | Batch on release | High |
| Kaggle - CRM + Sales + Opportunities | CSV | 340 MB | 8.9M | Batch on release | High |
| Kaggle - UCI Online Retail II | CSV | 180 MB | 1.1M | Static with periodic refresh | Medium |
| Kaggle - Consumer Complaints (supporting) | CSV | 500 MB | 3.6M | Batch on release | High |
| Mock CRM API | API (JSON, paginated) | 200-350 MB per full daily pull | 3.5M full pull; 20k-120k delta/run | 15-min incremental + nightly reconcile | High |
| Local Dummy JSON Drops | JSON files | 50 MB-5 GB per drop | 100k-2.0M per drop | Ad hoc / unknown | High |

## 1) Kaggle - Customer 360 Bundle

| Attribute | Value |
|---|---|
| Source Name | Kaggle - Customer 360 Bundle (customers.csv, orders.csv, support_tickets.csv, clickstream.csv) |
| Type | CSV |
| Estimated Size | ~480 MB compressed; ~1.1 GB uncompressed |
| Data Volume (rows) | customers: ~1.8M; orders: ~7.5M; support_tickets: ~2.1M; clickstream: ~14.2M; total: ~25.6M |
| Schema characteristics | Mixed schemas: customers is medium-wide (20-30 cols), orders/support medium (10-25 cols), clickstream narrow event model (8-12 cols); flat CSV; no nesting |
| Key fields | customer_id, order_id, ticket_id, session_id; candidate event key: (session_id, event_timestamp, page_url) |
| Known issues | Missing email/country values; duplicate customer_id across snapshots; inconsistent timestamp timezone labels; clickstream bot/noise sessions; malformed UTF-8 in free text |
| Refresh pattern | Static Kaggle snapshots; ingested in batch when new release is published |
| Ingestion complexity | High - multiple interrelated files, very high event volume, cross-file key validation, and strict timestamp harmonization needed |

## 2) Kaggle - CRM + Sales + Opportunities

| Attribute | Value |
|---|---|
| Source Name | Kaggle - CRM + Sales + Opportunities (accounts.csv, products.csv, opportunities.csv, sales_pipeline.csv, teams.csv) |
| Type | CSV |
| Estimated Size | ~340 MB compressed; ~760 MB uncompressed |
| Data Volume (rows) | accounts: ~1.2M; products: ~450k; opportunities: ~2.7M; sales_pipeline: ~3.9M; teams: ~650k; total: ~8.9M |
| Schema characteristics | Mostly medium-wide business tables (15-40 cols), denormalized text-heavy dimensions, flat CSV |
| Key fields | account_id, product_id, opp_id, deal_id, owner_id; candidate relationship keys: (account_id, opp_id), (owner_id, team_id) |
| Known issues | Mixed casing in categorical values; duplicate opportunities after status updates; inconsistent currency formatting; missing owner_id for orphan deals; ambiguous stage taxonomy |
| Refresh pattern | Static snapshot ingest with release-based batch updates |
| Ingestion complexity | High - cross-table referential consistency checks, currency normalization requirements, and high duplicate risk on sales entities |

## 3) Kaggle - UCI Online Retail II

| Attribute | Value |
|---|---|
| Source Name | Kaggle - UCI Online Retail II (online_retail_transactions.csv) |
| Type | CSV |
| Estimated Size | ~180 MB compressed; ~420 MB uncompressed |
| Data Volume (rows) | ~1.1M transactions |
| Schema characteristics | Narrow transactional schema (8-12 cols), flat CSV, high-cardinality invoice and stock fields |
| Key fields | invoice_no, stock_code, customer_id, invoice_date; candidate line-level key: (invoice_no, stock_code, invoice_date) |
| Known issues | Negative quantity/price for returns and corrections; missing customer_id; duplicate invoice lines; mixed locale formatting in numeric fields |
| Refresh pattern | Primarily static; occasional replacement snapshots |
| Ingestion complexity | Medium - single-file ingest is simple, but transaction anomalies and duplicate line handling increase validation load |

## 4) Kaggle - Consumer Complaints (Supporting)

| Attribute | Value |
|---|---|
| Source Name | Kaggle - Consumer Complaints (complaints.csv) |
| Type | CSV |
| Estimated Size | ~500 MB compressed; ~1.3 GB uncompressed |
| Data Volume (rows) | ~3.6M complaints |
| Schema characteristics | Wide semi-structured text schema (20-45 cols), flat CSV with long text columns |
| Key fields | complaint_id, customer_id (when present), submitted_date, issue_category |
| Known issues | High null rates in customer-linked fields; duplicate complaint narratives; inconsistent issue taxonomy over time; long-text encoding artifacts |
| Refresh pattern | Batch on release; not guaranteed regular cadence |
| Ingestion complexity | High - large text payloads, sparse identity keys, and taxonomy drift require stricter profiling and validation gates |

## 5) Mock CRM API

| Attribute | Value |
|---|---|
| Source Name | Mock CRM API (accounts, contacts, opportunities, activities endpoints) |
| Type | API (JSON, paginated) |
| Estimated Size | ~200-350 MB per full daily pull; ~5-30 MB per incremental run |
| Data Volume (rows) | ~3.5M objects full pull; ~20k-120k objects per incremental run |
| Schema characteristics | Nested JSON objects/arrays, sparse optional attributes, endpoint-specific schema variation |
| Key fields | account_id, contact_id, opportunity_id, activity_id, updated_at cursor, external_customer_ref |
| Known issues | 429 throttling/rate limits; duplicate page retrieval on retry; occasional null IDs in non-prod records; enum value drift and casing drift; late-arriving updates |
| Refresh pattern | 15-minute incremental (cursor-based) plus nightly full reconciliation |
| Ingestion complexity | High - requires robust pagination state, retry with backoff, idempotent upsert keys, watermark management, and schema drift handling |

## 6) Local Dummy JSON Drops

| Attribute | Value |
|---|---|
| Source Name | Local dummy JSON files (developer and QA drops) |
| Type | JSON files |
| Estimated Size | ~50 MB to ~5 GB per drop |
| Data Volume (rows) | ~100k to ~2.0M records per drop |
| Schema characteristics | Highly variable: nested arrays, optional nested objects, inconsistent field naming, mixed primitive types |
| Key fields | Often inconsistent; expected identifiers include customer_id, external_id, event_id; candidate fallback key: (source_file, row_index) |
| Known issues | Missing mandatory IDs; duplicated records across files; inconsistent timestamp formats; mixed numeric/string typing; null-heavy nested objects |
| Refresh pattern | Ad hoc/manual; often unknown ahead of arrival |
| Ingestion complexity | High - schema unpredictability, weak key guarantees, and non-deterministic delivery pattern require strict contract checks and quarantine paths |

## Cross-Source Challenges

| Challenge | Concrete Risk | Operational Impact | Decision Support Signal |
|---|---|---|---|
| Schema mismatch | Same business attributes appear with different names/types (for example, customer_id vs CustomerID vs external_customer_ref) | Broken joins, failed loads, inconsistent downstream metrics | Maintain source-level contract registry and enforce pre-ingestion schema checks |
| Entity resolution (customer identity problem) | No universal customer key across Kaggle CSV, API, and local JSON | Fragmented customer 360 view and duplicate customer profiles | Use deterministic matching keys first, then probabilistic matching with confidence thresholds |
| Data duplication risks | Overlap across snapshots, API retries, and multi-file exports causes repeated records | Double counting in facts and unstable aggregates | Apply source-specific dedup keys and ingestion idempotency tokens |
| Temporal inconsistencies | Event timestamps and update timestamps use mixed timezones and mixed precision | Incorrect sequence logic and stale-vs-current ambiguity | Standardize to UTC at ingestion boundary and retain original timestamp fields |

## Assumptions and Unknowns

### Assumptions

- Kaggle datasets remain snapshot-based and are ingested as batch drops.
- API supports stable pagination and a usable incremental cursor (updated_at or equivalent).
- Raw landing zone can store both compressed originals and normalized extract artifacts.
- Source-level metadata logging (source, extract_time, row_count, checksum) is mandatory.

### Unknowns

- Final contractual key for cross-source customer identity resolution is not confirmed.
- Exact release cadence and retention policy for each Kaggle dataset are not confirmed.
- API SLA, hard rate limits, and backfill limits are not confirmed.
- JSON drop governance (who produces files, naming rules, arrival windows) is not confirmed.
- Field-level PII classification and masking requirements are not yet finalized.
