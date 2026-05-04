# Architecture — {{PROJECT_NAME}}

> Scaffolded from `fabric-project-context-template` {{TEMPLATE_VERSION}}.
> Update this file when stack, boundaries, or invariants change.

## Stack

| Layer | Technology | Role |
|---|---|---|
| Compute | Microsoft Fabric Notebooks (PySpark 3.4+) | All transformation logic |
| Storage | OneLake + Delta Lake | Canonical storage for all medallion layers |
| Lakehouses | `{{LAKEHOUSE_PREFIX}}_landing`, `{{LAKEHOUSE_PREFIX}}_bronze`, `{{LAKEHOUSE_PREFIX}}_silver`, `{{LAKEHOUSE_PREFIX}}_gold` | One per medallion layer |
| Orchestration | Fabric Pipelines | Drives Bronze ingestion. Silver/Gold use orchestrator notebooks. |
| Secrets | Azure Key Vault via `notebookutils.credentials.getSecret()` | API keys, PATs, connection strings |
| Config | Fabric Variable Library | Environment-specific values (workspace IDs, lakehouse GUIDs, connection strings per env) |
| Logging | Custom `nb_utils_logging_framework` writing to control tables | Pipeline execution tracking |
| Source control | GitHub | Git integration with each Fabric workspace |
| CI/CD | GitHub Actions + `fabric-cli` | Workspace sync, Test/Prod promotion. Replaces Fabric GUI Deployment Pipeline. |
| Coding agent | Claude Code | `CLAUDE.md` entry point at repo root |
| Consumer | Power BI semantic models | Reads gold-layer tables |

## System boundaries

- **`workspace-assets/`** — Fabric-synced. Editing here propagates to the Fabric workspace on next push. Anything that should NOT live in Fabric does not belong here.
  - **`workspace-assets/00 Notebooks/01 Landing/`** — owns landing-layer ingestion (raw file landing, optional layer).
  - **`workspace-assets/00 Notebooks/02 Bronze/`** — owns bronze ingestion. One executor notebook per source system.
  - **`workspace-assets/00 Notebooks/03 Silver/`** — owns silver conformance. Per-system framework + orchestrator + entity notebooks.
  - **`workspace-assets/00 Notebooks/04 Gold/`** — owns gold facts and dims. Per-system framework + orchestrator + entity notebooks.
  - **`workspace-assets/00 Notebooks/05 Utility/`** — owns shared utilities (logging framework, control table writer, helpers).
  - **`workspace-assets/02 Pipelines/`** — owns Fabric Pipeline JSON definitions.
  - **`workspace-assets/{{LAKEHOUSE_PREFIX}}_*.Lakehouse/`** — owns lakehouse definitions and shortcut configurations.
- **`project/`** — local-only. Committed to git, never synced to Fabric.
  - **`project/context/`** — this directory. Six-file context system. Read by every Claude Code session before any change.
  - **`project/requests/`** — REQ task queue. State machine: `queue/` → `working/` → `done/` (or `blocked/`).
  - **`project/docs/`** — implementation guides, API documentation, DBML diagrams, implementation reports. (Optional, present if scaffolded with docs toggle.)
  - **`project/scripts/`** — verification and diagnostic scripts. Not deployed to Fabric. (Optional.)
- **`.github/workflows/`** — GitHub Actions for CI/CD. Owns scheduled Fabric workspace syncs and environment promotion logic.

## Storage model

- **`{{LAKEHOUSE_PREFIX}}_landing`** — raw files (CSV, JSON, Parquet) and lightly-typed Delta tables for sources that land files. Optional layer. Skip this paragraph if landing was not enabled at scaffold time.
- **`{{LAKEHOUSE_PREFIX}}_bronze`** — raw data with ingestion metadata. One Delta table per source object. Partitioned by `etl_batch_date`.
- **`{{LAKEHOUSE_PREFIX}}_silver`** — cleansed, conformed. Business rules applied. Idempotent. Deduplicated on business keys.
- **`{{LAKEHOUSE_PREFIX}}_gold`** — facts and dimensions. SCD Type 1 or Type 2 as documented per table. Read by Power BI semantic models.
- **Control tables** — pipeline execution tracking. Live in `{{LAKEHOUSE_PREFIX}}_bronze` (or a dedicated utils lakehouse if the project has one). One row per pipeline run.

## Authentication and access model

- **Workspace identity** — Fabric workspace identity is granted **Key Vault Secrets User** on the project's Azure Key Vault. All notebook secret reads go through `notebookutils.credentials.getSecret(<vault>, <secret>)`.
- **Service principal** (CI/CD) — a dedicated service principal authenticates GitHub Actions to Fabric. Its PAT or client secret lives in GitHub Actions secrets, never in the repo.
- **Source system credentials** — API keys and tokens live in Key Vault. Notebooks read them at runtime. Do not paste them into notebook cells, even temporarily.

<!-- IF MULTI_DEV -->
## Multi-developer model

- Each developer has a personal Fabric workspace and a `dev/<name>` git branch (e.g., `dev/{{PRIMARY_DEVELOPER}}`).
- Each personal workspace has its own lakehouse GUIDs.
- PRs from `dev/<name>` to `main` translate the GUIDs back to the Test workspace's GUIDs. The translation is automated by the project's PR skill.
- `main` represents the Test workspace.
- Promotion `main` → Prod runs via `.github/workflows/promote-to-prod.yml`.
<!-- END IF -->

## Invariants

The codebase must never violate these rules. Any violation is a defect.

1. **Pipelines are idempotent.** Re-running with the same logical inputs produces the same outputs.
2. **No `inferSchema=True` in production reads.** All schemas defined explicitly.
3. **No hardcoded environment values.** Workspace IDs, lakehouse GUIDs, connection strings come from Variable Library. Secrets come from Key Vault.
4. **`spark` is auto-available in Fabric.** Do not call `SparkSession.builder.getOrCreate()`.
5. **Bronze and Silver Delta tables include the `etl_*` metadata columns** as defined in `data-conventions.md`.
6. **No `.count()` or `.collect()` inside loops.** Use single-pass PySpark aggregations.
7. **Composite keys cast numeric IDs to `int` before `string`** to strip `.0` suffixes that PySpark introduces.
8. **MERGE source DataFrames are deduplicated on business keys** before the MERGE call.
9. **ZORDER excludes generated columns** — Delta cannot collect statistics on them.
10. **No `dbutils`, `%sql` magic, or other Databricks patterns.** This is Fabric. Use `notebookutils`.
11. [Add project-specific invariants discovered during the build. Examples: "all RentManager dates parse with `MM/dd/yyyy`", "StorEdge `entity_level` always specified explicitly".]
