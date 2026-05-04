# Data Conventions — {{PROJECT_NAME}}

Naming and structural rules for this project. Inherits the canonical conventions from `fabric-project-context-template` and adds project-specific extensions where needed.

## Lakehouses

| Layer | Name | Purpose |
|---|---|---|
| Landing | `{{LAKEHOUSE_PREFIX}}_landing` | Raw landed files. (Remove this row if landing was not enabled.) |
| Bronze | `{{LAKEHOUSE_PREFIX}}_bronze` | Raw + ingestion metadata. One Delta table per source object. |
| Silver | `{{LAKEHOUSE_PREFIX}}_silver` | Cleansed, conformed, business-rule-applied. Idempotent. |
| Gold | `{{LAKEHOUSE_PREFIX}}_gold` | Curated facts and dimensions. Read by Power BI. |

## Notebook naming

`nb_{layer}_{system}_{purpose}` where:

- `{layer}`: `landing`, `bronze`, `silver`, `gold`, or `utils`.
- `{system}`: source system identifier in lowercase. Project's known systems: [list source systems here, e.g., `storedge`, `rent_manager`, `monday`, `semantic_model`].
- `{purpose}`: function of the notebook. Common: `framework`, `orchestrator`, `runner`, or a specific entity.

Examples for {{PROJECT_NAME}}:

- `nb_bronze_[source]_[entity]` — bronze ingestion for one entity.
- `nb_silver_[source]_main_framework` — silver framework for a source.
- `nb_silver_[source]_orchestrator` — silver orchestrator.
- `nb_gold_[source]_[fact_or_dim]` — gold table builder.
- `nb_utils_logging_framework` — shared utility (no layer prefix; spans layers).

## Layer folder structure

```
workspace-assets/00 Notebooks/
├── 01 Landing/   {{IF_INCLUDE_LANDING}}
├── 02 Bronze/
├── 03 Silver/
├── 04 Gold/
└── 05 Utility/
```

Numbering is mandatory and conveys medallion flow. Do not rename to drop numbers.

## ETL metadata columns

Every Bronze and Silver Delta table includes:

| Column | Type | Purpose |
|---|---|---|
| `etl_loaded_at` | TIMESTAMP | `F.current_timestamp()` at write time. |
| `etl_batch_date` | DATE | Logical batch date. Most-common partition column. |
| `etl_source_system` | STRING | One of the project's source system identifiers. |
| `etl_pipeline_name` | STRING | The pipeline that produced the row. |
| `etl_record_status` | STRING | `active`, `superseded`, or `deleted`. |

Source-system extras (when applicable):

| Column | Type | Purpose |
|---|---|---|
| `etl_report_generated_at` | TIMESTAMP | Source report generation time (report-based ingestion). |
| `etl_report_id` | STRING | Source report identifier. |
| `etl_report_name` | STRING | Source report name. |

The `etl_` prefix is reserved. Do not create non-ETL columns with this prefix.

## Date and timestamp standards

- TIMESTAMP for points in time. DATE for calendar dates. STRING dates are not allowed in Silver or Gold.
- All timestamps stored in UTC.
- Source parsing — document each source's expected format here:
  - [Source 1]: `[format]`
  - [Source 2]: `[format]`

## Composite key construction

Numeric IDs cast to `int` then `string` to strip `.0`:

```python
F.concat(
    F.col("location_id").cast("int").cast("string"),
    F.lit("_"),
    F.col("property_id").cast("int").cast("string")
)
```

Composite keys built from float-typed source columns produce `1686.0_20.0` and fail to join against `1686_20`. Silent failure, zero match rate.

## Schema rules

- Explicit `StructType` for production reads.
- Column ordering: business keys → attributes → ETL metadata.
- snake_case column names. Source columns in PascalCase or with spaces are renamed at the bronze boundary.
- Reserved-character cleanup at bronze: space, comma, semicolon, period, parentheses, braces, brackets, equals, slash, newline, tab.

## Partitioning and ZORDER

- Bronze and Silver: partition by `etl_batch_date`.
- Gold facts: partition per access pattern (often `snapshot_date_key` for SCD Type 2).
- ZORDER on physical columns only. Never on generated columns.

## SCD

- Type 1: Delta `MERGE`. Overwrite. No history.
- Type 2: history-tracked. Required columns: `start_date`, `end_date`, `is_current`, `surrogate_key`.
- Always deduplicate the source DataFrame on business keys before MERGE.

## Idempotency

- Bronze/Silver date-partitioned: partition overwrite (`partitionOverwriteMode=dynamic`).
- Silver/Gold dims: MERGE on business keys.
- Silver/Gold facts: full overwrite of affected partition or MERGE on natural key + grain.

Non-idempotent pipelines are a defect.

## Control table schema

| Column | Type | Notes |
|---|---|---|
| `pipeline_run_id` | STRING (UUID) | Unique per run. |
| `pipeline_name` | STRING | Matches `etl_pipeline_name`. |
| `source_system` | STRING | One of the project's source system identifiers. |
| `layer` | STRING | `landing`, `bronze`, `silver`, `gold`, `utils`. |
| `start_time` | TIMESTAMP | UTC. |
| `end_time` | TIMESTAMP | UTC. NULL while running. |
| `duration_seconds` | DOUBLE | Computed at end. |
| `records_processed` | LONG | Optional. |
| `status` | STRING | `running`, `success`, `failed`, `skipped`. |
| `error_message` | STRING | NULL on success. |

Control table lives in `{{LAKEHOUSE_PREFIX}}_bronze` (or a dedicated utils lakehouse if added later).

## Project-specific conventions

[Add rules unique to {{PROJECT_NAME}}. Document the *why* alongside each rule. Examples to refine or remove:]

- [Source 1]-specific: ...
- [Source 2]-specific: ...
