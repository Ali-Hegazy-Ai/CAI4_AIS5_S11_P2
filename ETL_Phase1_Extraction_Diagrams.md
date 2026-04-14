# Phase 1: Source Identification & Extraction Diagrams

---

## 1. SOURCE LANDSCAPE DIAGRAM

```mermaid
graph TB
    subgraph Kaggle["KAGGLE DATASETS"]
        K1["📊 Customer 360<br/>customers.csv<br/>orders.csv<br/>support_tickets.csv<br/>clickstream.csv<br/><br/>PRIMARY | Auto Download"]
        K2["📊 UCI Online Retail II<br/>online_retail_transactions.csv<br/><br/>PRIMARY | Auto Download"]
        K3["📊 Consumer Complaints<br/>complaints.csv<br/><br/>OPTIONAL | Auto Download"]
    end
    
    subgraph Local["LOCAL / SHARED STORAGE"]
        L1["🗂️ CRM + Sales Systems<br/>accounts.csv<br/>products.csv<br/>opportunities.csv<br/>sales_pipeline.csv<br/>teams.csv<br/><br/>PRIMARY | File Share / Git"]
    end
    
    subgraph Ownership["OWNERSHIP & ACCESS"]
        O1["Kaggle API Key<br/>Authenticated Downloads"]
        O2["Shared Drive<br/>Local Repository"]
    end
    
    K1 -.→ O1
    K2 -.→ O1
    K3 -.→ O1
    L1 -.→ O2
    
    style K1 fill:#e1f5ff
    style K2 fill:#e1f5ff
    style K3 fill:#fff9c4
    style L1 fill:#f3e5f5
```

---

## 2. DATA FLOW DIAGRAM (LEVEL 0 - HIGH LEVEL)

```mermaid
graph LR
    A["📂 DATA SOURCES<br/>Kaggle + Local CSV"]
    B["🔄 EXTRACTION LAYER<br/>Read, Validate, Check"]
    C["✅ VALIDATION<br/>CHECKPOINT"]
    D["💾 RAW LANDING ZONE<br/>Azure Blob / Local Data Lake"]
    
    A -->|Ingestion Trigger| B
    B -->|Pass Schema Check| C
    B -->|Fail Schema Check| E["❌ QUARANTINE"]
    C -->|Validated Records| D
    E -->|Bad Data| F["🚨 ERROR LOG"]
    
    style A fill:#e3f2fd
    style B fill:#fff9c4
    style C fill:#c8e6c9
    style D fill:#f3e5f5
    style E fill:#ffccbc
    style F fill:#ffcdd2
```

---

## 3. DATA FLOW DIAGRAM (LEVEL 1 - DETAILED STEPS)

```mermaid
graph TD
    S1["Source Discovery<br/>List available CSV files"]
    S2["File Acquisition<br/>Download or Read from Local"]
    S3["Schema Inference<br/>Parse headers & types"]
    S4["Row Count Validation<br/>Verify non-empty"]
    S5["Field Validation<br/>Check required fields exist"]
    S6["Type Inference<br/>Detect column data types"]
    S7["Metadata Logging<br/>Record stats & lineage"]
    S8["Load to Raw Storage<br/>Copy to /raw/{source}/{date}"]
    
    S1 -->|CSV List| S2
    S2 -->|File Handle| S3
    S3 -->|Headers| S4
    S4 -->|Record Count| S5
    S5 -->|Field Names| S6
    S6 -->|Column Schemas| S7
    S7 -->|Metadata| S8
    
    S4 -->|⚠️ Empty| Q1["QUARANTINE<br/>Empty File"]
    S5 -->|⚠️ Missing Field| Q2["QUARANTINE<br/>Invalid Schema"]
    S6 -->|⚠️ Type Mismatch| Q3["QUARANTINE<br/>Type Conflict"]
    
    Q1 --> LOG["🔴 Log Error"]
    Q2 --> LOG
    Q3 --> LOG
    
    S8 --> SUCCESS["✅ Extraction Complete"]
    
    style S1 fill:#e3f2fd
    style S2 fill:#e3f2fd
    style S3 fill:#fff9c4
    style S4 fill:#fff9c4
    style S5 fill:#fff9c4
    style S6 fill:#fff9c4
    style S7 fill:#c8e6c9
    style S8 fill:#c8e6c9
    style Q1 fill:#ffccbc
    style Q2 fill:#ffccbc
    style Q3 fill:#ffccbc
    style LOG fill:#ffcdd2
    style SUCCESS fill:#a5d6a7
```

---

## 4. EXTRACTION PIPELINE DIAGRAM

```mermaid
graph TD
    T0["⏱️ TRIGGER<br/>Manual / Scheduled"]
    T1["🔍 DETECT NEW FILES<br/>Scan source folders for *.csv"]
    T2["📖 READ CSV<br/>Load into memory with encoding check"]
    T3["🔎 VALIDATE SCHEMA<br/>Compare against expected columns"]
    T4["✔️ VALIDATE ROWS<br/>Check for required fields not null"]
    T5["📊 INFER METADATA<br/>Row count, file size, hash"]
    T6["📝 LOG METADATA<br/>Write to ingestion logs"]
    T7["📤 COPY TO RAW<br/>Move to /raw/{source}/YYYY/MM/DD/{file}_{timestamp}"]
    T8["✅ UPDATE STATUS<br/>Mark as 'LOADED'"]
    
    T0 --> T1
    T1 --> T2
    T2 --> T3
    T3 -->|Valid| T4
    T3 -->|Invalid| BADF["❌ SCHEMA ERROR<br/>Expected cols missing"]
    T4 -->|Valid| T5
    T4 -->|Invalid| BADT["❌ TYPE ERROR<br/>Type mismatch in row"]
    T5 --> T6
    T6 --> T7
    T7 --> T8
    
    BADF --> MOVEQ1["Move to /quarantine/{file}"]
    BADT --> MOVEQ2["Move to /quarantine/{file}"]
    MOVEQ1 --> LOGX["Log error to ingestion_log.csv"]
    MOVEQ2 --> LOGX
    
    T8 --> DONE["✅ EXTRACTION COMPLETE"]
    
    style T0 fill:#e3f2fd
    style T1 fill:#fff9c4
    style T2 fill:#fff9c4
    style T3 fill:#fff9c4
    style T4 fill:#fff9c4
    style T5 fill:#c8e6c9
    style T6 fill:#c8e6c9
    style T7 fill:#c8e6c9
    style T8 fill:#a5d6a7
    style BADF fill:#ffccbc
    style BADT fill:#ffccbc
    style MOVEQ1 fill:#ffccbc
    style MOVEQ2 fill:#ffccbc
    style LOGX fill:#ffcdd2
    style DONE fill:#a5d6a7
```

---

## 5. SYSTEM ARCHITECTURE DIAGRAM

```mermaid
graph TB
    subgraph Sources["SOURCE SYSTEMS"]
        K["🌐 KAGGLE<br/>API Access"]
        L["🗂️ LOCAL STORAGE<br/>Shared Drive / Git"]
    end
    
    subgraph Extract["EXTRACTION ENGINE"]
        P["🐍 Python Runner<br/>OR<br/>⚙️ ADF Pipeline"]
    end
    
    subgraph Process["PROCESSING & VALIDATION"]
        F["File Reader<br/>CSV Parser"]
        V["Schema Validator<br/>Type Checker"]
        M["Metadata Logger<br/>Lineage Tracker"]
    end
    
    subgraph Storage["RAW STORAGE"]
        B["💾 Local Data Lake<br/>OR<br/>☁️ Azure Blob Storage"]
    end
    
    subgraph Archive["ARCHIVE & QUARANTINE"]
        Q["🚫 Quarantine Zone<br/>Invalid Files"]
        A["📦 Archive Zone<br/>Completed Ingestionsn"]
    end
    
    K -->|Download CSV| Extract
    L -->|Read CSV| Extract
    Extract --> Process
    Process -->|Valid| B
    Process -->|Invalid| Q
    B --> A
    
    style Sources fill:#e3f2fd
    style Extract fill:#fff9c4
    style Process fill:#fff9c4
    style Storage fill:#c8e6c9
    style Archive fill:#f3e5f5
    style K fill:#e1f5ff
    style L fill:#e1f5ff
    style B fill:#a5d6a7
    style Q fill:#ffccbc
    style A fill:#d1c4e9
```

---

## 6. DATA CONTRACT / SCHEMA BOUNDARY DIAGRAM

```mermaid
graph TD
    subgraph CUST["CUSTOMER 360 - Kaggle"]
        C["customers.csv<br/>━━━━━━━━━━━━<br/>✓ customer_id<br/>✓ name<br/>✓ email<br/>✓ country<br/>✓ register_date"]
        O["orders.csv<br/>━━━━━━━━━━━━<br/>✓ order_id<br/>✓ customer_id<br/>✓ order_date<br/>✓ amount<br/>? shipping_date"]
        ST["support_tickets.csv<br/>━━━━━━━━━━━━<br/>✓ ticket_id<br/>✓ customer_id<br/>? resolution_date<br/>✓ status"]
        CL["clickstream.csv<br/>━━━━━━━━━━━━<br/>✓ session_id<br/>✓ customer_id<br/>✓ page<br/>✓ timestamp"]
    end
    
    subgraph CRM["CRM + SALES - Local"]
        AC["accounts.csv<br/>━━━━━━━━━━━━<br/>✓ account_id<br/>✓ company_name<br/>✓ industry<br/>✓ owner_id"]
        PR["products.csv<br/>━━━━━━━━━━━━<br/>✓ product_id<br/>✓ product_name<br/>✓ price<br/>✓ category"]
        OP["opportunities.csv<br/>━━━━━━━━━━━━<br/>✓ opp_id<br/>✓ account_id<br/>✓ amount<br/>✓ stage"]
        SP["sales_pipeline.csv<br/>━━━━━━━━━━━━<br/>✓ deal_id<br/>✓ account_id<br/>✓ close_date<br/>? notes"]
        TM["teams.csv<br/>━━━━━━━━━━━━<br/>✓ owner_id<br/>✓ name<br/>✓ region"]
    end
    
    subgraph RETAIL["UCI ONLINE RETAIL - Kaggle"]
        TX["online_retail_transactions.csv<br/>━━━━━━━━━━━━<br/>✓ invoice_no<br/>✓ stock_code<br/>✓ quantity<br/>✓ invoice_date<br/>✓ price"]
    end
    
    subgraph OPT["CONSUMER COMPLAINTS - Kaggle (Optional)"]
        CMP["complaints.csv<br/>━━━━━━━━━━━━<br/>✓ complaint_id<br/>✓ customer_id<br/>? resolution_time<br/>✓ category"]
    end
    
    style CUST fill:#e3f2fd
    style CRM fill:#f3e5f5
    style RETAIL fill:#e3f2fd
    style OPT fill:#fff9c4
    style C fill:#bbdefb
    style O fill:#bbdefb
    style ST fill:#bbdefb
    style CL fill:#bbdefb
    style AC fill:#e1bee7
    style PR fill:#e1bee7
    style OP fill:#e1bee7
    style SP fill:#e1bee7
    style TM fill:#e1bee7
    style TX fill:#bbdefb
    style CMP fill:#ffe0b2
```

Legend:
- **✓** = Required (non-nullable)
- **?** = Optional (nullable)

---

## 7. FILE LIFECYCLE DIAGRAM

```mermaid
stateDiagram-v2
    [*] --> INCOMING: CSV arrives at source
    
    INCOMING --> DETECTED: File discovered by scanner
    
    DETECTED --> READING: Begin read operation
    
    READING --> SCHEMA_CHECK: Parse & infer schema
    
    SCHEMA_CHECK --> VALID_SCHEMA: Headers match contract
    SCHEMA_CHECK --> INVALID_SCHEMA: Headers missing/wrong
    
    VALID_SCHEMA --> ROWS_CHECK: Validate data integrity
    
    ROWS_CHECK --> VALID_ROWS: All required fields present
    ROWS_CHECK --> INVALID_ROWS: Missing/bad data found
    
    VALID_ROWS --> METADATA_LOG: Log extraction stats
    
    METADATA_LOG --> STORED: File copied to /raw<br/>/raw/{source}/YYYY/MM/DD<br/>{filename}_{timestamp}.csv
    
    STORED --> ARCHIVED: Move to archive zone<br/>(keep for audit)
    
    INVALID_SCHEMA --> QUARANTINE: Move to /quarantine<br/>/quarantine/{filename}_{timestamp}
    
    INVALID_ROWS --> QUARANTINE
    
    QUARANTINE --> ERROR_LOGGED: Log failure reason
    
    ERROR_LOGGED --> [*]
    
    ARCHIVED --> [*]
    
    note right of STORED
        ✓ Extraction Complete
        • Row count logged
        • File hash logged
        • Timestamp recorded
    end note
    
    note right of QUARANTINE
        ✗ Extraction Failed
        • Error message logged
        • File preserved
        • Requires manual review
    end note
```

---

## 8. ERROR HANDLING FLOW DIAGRAM

```mermaid
graph TD
    START["📂 File Detected"]
    
    START --> OPENCHK{"Can file<br/>be opened?"}
    
    OPENCHK -->|Yes| ENCODCHK{"Valid<br/>encoding?"}
    OPENCHK -->|No| ERR1["🔴 FILE_NOT_FOUND<br/>or<br/>PERMISSION_DENIED"]
    
    ENCODCHK -->|Yes| SCHEMACHK{"Schema<br/>valid?"}
    ENCODCHK -->|No| ERR2["🔴 ENCODING_ERROR<br/>(UTF-8 expected)"]
    
    SCHEMACHK -->|Yes| ROWCHK{"All required<br/>fields<br/>populated?"}
    SCHEMACHK -->|No| ERR3["🔴 SCHEMA_MISMATCH<br/>Expected columns:<br/>customer_id, date, amount"]
    
    ROWCHK -->|Yes| TYPECHK{"Column types<br/>match<br/>schema?"}
    ROWCHK -->|No| ERR4["🔴 NULL_CONSTRAINT<br/>Required field is null"]
    
    TYPECHK -->|Yes| SUCCESS["✅ VALID<br/>Load to /raw"]
    TYPECHK -->|No| ERR5["🔴 TYPE_MISMATCH<br/>Expected INT, got VARCHAR"]
    
    ERR1 --> QUAR["📤 Move to<br/>/quarantine/{source}/"]
    ERR2 --> QUAR
    ERR3 --> QUAR
    ERR4 --> QUAR
    ERR5 --> QUAR
    
    QUAR --> LOGFILE["📝 Write to<br/>ingestion_errors.log<br/>━━━━━━━━━━━━<br/>timestamp | source | error_code<br/>error message | file path<br/>━━━━━━━━━━━━"]
    
    LOGFILE --> ALERT["🚨 (Optional)<br/>Send notification<br/>to data team"]
    
    SUCCESS --> RAW["📥 Copy to<br/>/raw/{source}/YYYY/MM/DD/"]
    
    RAW --> LOGPATH["📝 Write to<br/>ingestion_success.log"]
    
    LOGPATH --> DONE["✅ Complete"]
    
    ALERT --> MANUAL["👤 Manual Review<br/>& Fix Required"]
    
    MANUAL --> RERUN["🔄 Re-run<br/>Extraction"]
    
    RERUN --> START
    
    style START fill:#e3f2fd
    style OPENCHK fill:#fff9c4
    style ENCODCHK fill:#fff9c4
    style SCHEMACHK fill:#fff9c4
    style ROWCHK fill:#fff9c4
    style TYPECHK fill:#fff9c4
    style SUCCESS fill:#a5d6a7
    style ERR1 fill:#ffccbc
    style ERR2 fill:#ffccbc
    style ERR3 fill:#ffccbc
    style ERR4 fill:#ffccbc
    style ERR5 fill:#ffccbc
    style QUAR fill:#ffccbc
    style LOGFILE fill:#ffcdd2
    style ALERT fill:#ffcdd2
    style RAW fill:#a5d6a7
    style LOGPATH fill:#c8e6c9
    style DONE fill:#a5d6a7
    style MANUAL fill:#fff9c4
    style RERUN fill:#fff9c4
```

---

## 9. INGESTION TRIGGER DIAGRAM

```mermaid
graph TB
    subgraph MANUAL["MANUAL TRIGGER"]
        M1["👤 User Action<br/>Run extraction script"]
        M2["python extract.py<br/>--source=customers<br/>--date=2025-04-14"]
    end
    
    subgraph SCHEDULED["SCHEDULED TRIGGER"]
        S1["⏰ Scheduler<br/>(Airflow / ADF / Cron)"]
        S2["Execution Time<br/>Daily: 00:00 UTC<br/>Weekly: Monday<br/>On-Demand: API Call"]
    end
    
    subgraph ENGINE["EXTRACTION ENGINE"]
        E1["Load Trigger Config"]
        E2["Verify Credentials<br/>(Kaggle API Key / File Access)"]
        E3["Initiate Extraction Pipeline"]
    end
    
    subgraph PIPELINE["EXTRACTION PIPELINE"]
        P1["🔍 Detect Files"]
        P2["📖 Read CSV"]
        P3["✔️ Validate"]
        P4["📤 Load to Raw"]
    end
    
    subgraph MONITOR["MONITORING & LOGGING"]
        L1["✅ Success: ingestion_success.log"]
        L2["❌ Failure: ingestion_errors.log"]
        L3["📊 Metrics: rows_loaded, duration, status"]
    end
    
    M1 --> M2
    S1 --> S2
    
    M2 -->|Trigger Signal| E1
    S2 -->|Trigger Signal| E1
    
    E1 --> E2
    E2 -->|Credentials OK| E3
    E2 -->|Credentials Fail| CRED_ERR["🔴 AUTH_ERROR<br/>Cannot access source"]
    
    E3 --> P1
    P1 --> P2
    P2 --> P3
    P3 -->|Valid| P4
    P3 -->|Invalid| QUAR_ERR["🔴 Quarantine File"]
    
    P4 --> L1
    QUAR_ERR --> L2
    CRED_ERR --> L2
    
    L1 --> L3
    L2 --> L3
    
    L3 --> REPORT["📈 Generate Ingestion Report"]
    
    style M1 fill:#e3f2fd
    style M2 fill:#e3f2fd
    style S1 fill:#e3f2fd
    style S2 fill:#e3f2fd
    style E1 fill:#fff9c4
    style E2 fill:#fff9c4
    style E3 fill:#fff9c4
    style P1 fill:#fff9c4
    style P2 fill:#fff9c4
    style P3 fill:#fff9c4
    style P4 fill:#c8e6c9
    style L1 fill:#a5d6a7
    style L2 fill:#ffcdd2
    style L3 fill:#c8e6c9
    style REPORT fill:#a5d6a7
    style CRED_ERR fill:#ffccbc
    style QUAR_ERR fill:#ffccbc
```

---

## Key Naming Conventions

```
Raw Storage Path Structure:
/raw/{source_name}/YYYY/MM/DD/

Examples:
/raw/customer_360/2025/04/14/customers_2025-04-14_143022.csv
/raw/customer_360/2025/04/14/orders_2025-04-14_143045.csv
/raw/crmn_sales/2025/04/14/accounts_2025-04-14_130500.csv
/raw/uci_retail/2025/04/14/online_retail_2025-04-14_090000.csv

Quarantine Path:
/quarantine/{source_name}/{filename}_{timestamp}_{error_code}.csv

Example:
/quarantine/customer_360/support_tickets_2025-04-14_143200_SCHEMA_MISMATCH.csv

Metadata Files:
ingestion_success.log
ingestion_errors.log
ingestion_stats_{YYYYMMDD}.json
```

---

## Metadata Fields to Log Per Extraction

```
source_name: string                    (customer_360, crmn_sales, etc.)
file_name: string                      (customers.csv)
extracted_timestamp: datetime          (2025-04-14 14:30:22)
extract_duration_seconds: integer      (2)
row_count: integer                     (50000)
file_size_bytes: integer               (2500000)
file_hash_md5: string                  (a1b2c3d4e5f6...)
schema_status: enum                    (VALID, INVALID, INFERRED)
row_validation_status: enum            (PASSED, FAILED, WARNINGS)
extraction_status: enum                (SUCCESS, QUARANTINED, ERROR)
error_code: string                     (NULL or SCHEMA_MISMATCH, TYPE_ERROR, etc.)
error_message: string                  (NULL or detailed error)
extracted_path: string                 (/raw/customer_360/2025/04/14/...)
quarantine_path: string                (NULL or /quarantine/...)
```

---

**END OF PHASE 1 EXTRACTION DIAGRAMS**
