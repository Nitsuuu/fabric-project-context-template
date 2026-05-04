# Data Conventions

Naming and structural rules the template enforces in *scaffolded projects*. These are the data-engineering equivalent of design tokens — every notebook, lakehouse, and pipeline in a scaffolded project follows them so the codebase stays legible across teams and projects.

These rules are scaffolded into each project's own `data-conventions.md`. The version here is canonical.

## Lakehouse naming

| Layer | Lakehouse name (default) | Purpose |
|---|---|---|
| Landing | `lh_landing` | Raw landed files (CSV, JSON, Parquet from sources). Optional layer; can be skipped if Bronze ingests directly. |
| Bronze | `lh_bronze` | Raw data with light typing and ingestion metadata. One Delta table per source object. |
| Silver | `lh_silver` | Cleansed, conformed, business-rule-applied data. Idempotent. |
| Gold | `lh_gold` | Curated facts and dimensions for analytics. Read by Power BI semantic models. |

The `lh_` prefix is configurable via `{{LAKEHOUSE_PREFIX}}` (default `lh`). All four lakehouses use the same prefix.

If a project has multiple gold lakehouses (e.g., one per business domain), use `lh_gold_<domain>` rather than dropping the prefix.

## Notebook naming

`nb_{layer}_{system}_{purpose}`

| Token | Values |
|---|---|
| `{layer}` | `landing`, `bronze`, `silver`, `gold`, `utils` |
| `{system}` | Source system identifier in lowercase (`storedge`, `rent_manager`, `monday`, `semantic_model`). For framework or shared notebooks, use `main` (e.g., `nb_silver_main_framework`). |
| `{purpose}` | What the notebook does. Common values: `framework`, `orchestrator`, `runner`, or a specific entity name (`facility`, `rent_roll`, `general_ledger`). |

Examples:

- `nb_landing_storedge` — landing-layer ingestion for StorEdge.
- `nb_bronze_rent_manager_location_4` — Rent Manager bronze for location 4.
- `nb_silver_storedge_orchestrator` — StorEdge silver-layer orchestrator.
- `nb_gold_rent_manager_main_framework` — gold-layer framework for Rent Manager.
- `nb_utils_logging_framework` — shared utility (no layer prefix because it spans layers).

The framework + executor pattern (see `architecture.md` of the scaffolded project) requires:

- One framework notebook per system per layer for Silver and Gold (e.g., `nb_silver_storedge_main_framework`).
- One orchestrator per system per layer for Silver and Gold.
- Bronze and Landing typically have no orchestrator — pipelines drive them.

## Layer folder structure

```
workspace-assets/00 Notebooks/
├── 01 Landing/
├── 02 Bronze/
├── 03 Silver/
├── 04 Gold/
└── 05 Utility/
```

Numbering at the layer level conveys medallion *flow* and is mandatory. Do not rename folders to drop numbers; do not add new top-level numbered folders without a major-version template change.

## ETL metadata columns

Every Bronze and Silver Delta table includes the following columns. Gold tables include them only on staging tables, not on final fact/dim tables.

| Column | Type | Purpose |
|---|---|---|
| `etl_loaded_at` | TIMESTAMP | When this row was written by the pipeline. `F.current_timestamp()` at write time. |
| `etl_batch_date` | DATE | The logical batch date. Used as a partition column on most tables. |
| `etl_source_system` | STRING | Source system identifier (`storedge`, `rent_manager`, etc.). |
| `etl_pipeline_name` | STRING | Pipeline that produced this row (`bronze_storedge_rent_roll`). |
| `etl_record_status` | STRING | `active`, `superseded`, or `deleted`. SCD-aware. |

Source-system-specific extras (when applicable):

| Column | Type | Purpose |
|---|---|---|
| `etl_report_generated_at` | TIMESTAMP | When the source report was generated (for report-based ingestion). |
| `etl_report_id` | STRING | Source report identifier. |
| `etl_report_name` | STRING | Source report name. |

The `etl_` prefix is reserved. Do not create non-ETL columns with this prefix.

## Date and timestamp standards

- **Storage**: TIMESTAMP for points in time, DATE for calendar dates. Never store dates as STRING in Silver or Gold.
- **Time zone**: UTC. Convert at the presentation layer, not in Silver/Gold.
- **Source parsing**:
  - RentManager CSV reports use `MM/dd/yyyy` (forward slashes).
  - StorEdge reports use ISO `yyyy-MM-dd`.
  - Always check source format explicitly; never rely on Spark date inference in production.

## Composite key construction

When concatenating numeric IDs into a string composite key, cast to `int` first to remove `.0` suffixes that PySpark introduces when columns are inferred as float:

```python
F.concat(
    F.col("location_id").cast("int").cast("string"),
    F.lit("_"),
    F.col("property_id").cast("int").cast("string")
)
```

A composite key produced from float-typed source columns will produce `1686.0_20.0`, which fails to join against the same key produced as int (`1686_20`). This is a high-impact silent failure: zero match rate, no error.

## Schema standards

- **Always define schemas explicitly** using `StructType` for production reads. Never use `inferSchema=True`.
- **Order columns deliberately**: business keys first, then attributes (alphabetical or grouped), then ETL metadata last.
- **Snake_case for all column names**. Source columns that arrive in PascalCase or with spaces are renamed at the bronze boundary.
- **No invalid Delta column characters**: space, comma, semicolon, period, parentheses, braces, brackets, equals, slash, newline, tab. Strip or replace at the bronze boundary.

## Partitioning and ZORDER

- **Partition** large tables by `etl_batch_date` (Bronze, Silver). Gold partitioning depends on access pattern (often `snapshot_date_key` for SCD Type 2 facts).
- **ZORDER** on physical columns only — never on generated columns. Delta cannot collect statistics on generated columns, and ZORDER fails silently.

## SCD conventions

- **SCD Type 1**: overwrite via Delta `MERGE`. No history.
- **SCD Type 2**: history-tracked. Required columns: `start_date` (TIMESTAMP), `end_date` (TIMESTAMP, NULL for current), `is_current` (BOOLEAN), `surrogate_key` (LONG, monotonically increasing).
- **Always deduplicate the source DataFrame on business keys before MERGE.** Delta MERGE fails on multiple source rows matching one target row.

## Idempotency

Every pipeline must be safe to re-run with the same logical inputs:

- **Date-partitioned Bronze**: partition overwrite (`partitionOverwriteMode=dynamic` + filter to the target partitions).
- **Silver/Gold dimensions**: Delta `MERGE` on business keys.
- **Silver/Gold facts**: full overwrite of the affected partition, or MERGE on natural key + grain.

Non-idempotent pipelines are a defect. There is no "fix forward" excuse — fix the pipeline.

## Control table conventions

Every pipeline writes a row to the control table at start and updates it at end with status, row counts, and duration. Schema (canonical):

| Column | Type | Notes |
|---|---|---|
| `pipeline_run_id` | STRING (UUID) | Unique per run. |
| `pipeline_name` | STRING | Matches `etl_pipeline_name`. |
| `source_system` | STRING | Matches `etl_source_system`. |
| `layer` | STRING | `landing`, `bronze`, `silver`, `gold`, `utils`. |
| `start_time` | TIMESTAMP | UTC. |
| `end_time` | TIMESTAMP | UTC, NULL until pipeline completes. |
| `duration_seconds` | DOUBLE | Computed at end. |
| `records_processed` | LONG | Optional, populated where applicable. |
| `status` | STRING | `running`, `success`, `failed`, `skipped`. |
| `error_message` | STRING | NULL on success. |

Control tables live in the `lh_utils` lakehouse if present, or `lh_bronze` otherwise. The template scaffolds the table definition in the chosen lakehouse.

## What this file does NOT cover

- Source-specific transformations (StorEdge `entity_level` quirks, RentManager date format edge cases). These belong in `project/docs/api/` of each scaffolded project.
- Spark configuration. Workload-specific; defaults are platform's defaults.
- Power BI semantic model conventions beyond "consume gold tables." Semantic model design lives in the BI team's documentation, not here.
