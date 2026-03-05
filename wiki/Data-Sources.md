# Data Sources

This page describes the two customer data sources used in this ETL project: a CRM CSV export and an Excel spreadsheet.

---

## Source 1: CRM Export (CSV)

### File Location
`data/raw/<filename>.csv`

### File Format
- Encoding: UTF-8
- Delimiter: comma (`,`)
- Header row: yes (first row)
- Date format: `YYYY-MM-DD`

### Expected Columns

| Column Name | Data Type | Example | Notes |
|---|---|---|---|
| `customer_id` | Integer | `1001` | Primary key — must not be null |
| `full_name` | String | `John Doe` | Full name, may include extra spaces |
| `email` | String | `john@example.com` | Lowercase preferred, not always valid |
| `phone` | String | `01012345678` | Inconsistent formats across records |
| `signup_date` | String / Date | `2023-05-10` | May be missing for older records |
| `country` | String | `Egypt` | May use abbreviations |
| `segment` | String | `Premium` | Values: `Premium`, `Standard`, `Basic` |

### Known Quality Issues
- Some `phone` values include dashes, spaces, or country code prefixes.
- `signup_date` may be blank for records imported before 2022.
- `segment` values are inconsistently cased (e.g. `premium`, `PREMIUM`, `Premium`).
- Duplicate `customer_id` values can appear when CRM records are exported incrementally.

---

## Source 2: Excel File

### File Location
`data/raw/<filename>.xlsx`

### File Format
- Sheet name: `Customers` (first sheet)
- Header row: yes (row 1)
- Date format: Excel serial number or `DD/MM/YYYY`

### Expected Columns

| Column Name | Data Type | Example | Notes |
|---|---|---|---|
| `CustomerID` | Integer | `1001` | Primary key — must not be null |
| `Name` | String | `John Doe` | May have leading/trailing spaces |
| `EmailAddress` | String | `John@Example.COM` | Mixed case |
| `PhoneNumber` | String | `+20-101-2345678` | May be formatted differently than CRM |
| `JoinDate` | Date / String | `10/05/2023` | DD/MM/YYYY format |
| `Country` | String | `EGY` | ISO 3166-1 alpha-3 or full name |
| `CustomerSegment` | String | `PREMIUM` | Uppercase variant of segment values |

### Known Quality Issues
- `JoinDate` may come through as an Excel serial number — ADF will need to convert it.
- `Country` uses 3-letter ISO codes inconsistently with the CRM which uses full country names.
- Some rows may have merged cells in Excel — these produce blank values in ADF.

---

## Overlap Between Sources

Both sources may contain records for the same customer (same `CustomerID` / `customer_id`). The ETL pipeline handles this by:

1. Deduplicating on `CustomerID` after merging.
2. Preferring the **CRM record** as the primary source when a conflict exists.
3. Falling back to the Excel record when the CRM record is missing a field.

---

## Adding New Source Files

1. Place the new file in `data/raw/` following the naming convention:
   - CRM: `crm_customers_YYYYMMDD.csv`
   - Excel: `excel_customers_YYYYMMDD.xlsx`
2. Update the ADF dataset (`ds_crm_source` or `ds_excel_source`) if the file name or schema has changed.
3. Run a test pipeline execution in **Debug** mode before triggering a full run.
4. Commit the new raw file and any updated ADF assets.

---

## File Naming Convention

| Source | Pattern | Example |
|---|---|---|
| CRM Export | `crm_customers_YYYYMMDD.csv` | `crm_customers_20240101.csv` |
| Excel | `excel_customers_YYYYMMDD.xlsx` | `excel_customers_20240101.xlsx` |
