# Project Overview — {{PROJECT_NAME}}

> Scaffolded from `fabric-project-context-template` {{TEMPLATE_VERSION}} on {{SCAFFOLD_DATE}}.
> Update this file as the project's scope evolves. Keep it factual and concrete.

## What this is

{{PROJECT_DESCRIPTION}}

[Replace this paragraph with a one-paragraph description: what the project does, the source systems it ingests, the consumer of the gold layer (Power BI semantic models, downstream apps, analytics teams), and the business outcome it enables. Be specific. Avoid vague language like "improves data quality" — say "reduces month-end close from 5 days to 2 days by automating GL extraction.".]

## Goals

[Numbered list. Each goal must be measurable, not aspirational. Examples below — replace with project-specific goals.]

1. Land {{primary source}} data in Bronze with end-to-end latency under [N] hours.
2. Conform [N] business entities into Silver with documented data quality rules.
3. Publish [N] gold-layer fact and dimension tables consumed by [downstream consumer].
4. [Goal 4]
5. [Goal 5]

## Core user flow

[Step-by-step, no gaps. Describe how data flows from source to consumer, including the human checkpoints.]

1. Source system ([source 1, source 2, ...]) produces data.
2. Ingestion notebooks under `workspace-assets/00 Notebooks/01 Landing/` (or `02 Bronze/` if no landing layer) run on schedule via Fabric pipelines.
3. Bronze tables are materialized with `etl_*` metadata columns.
4. Silver notebooks under `03 Silver/` apply business rules and conform entities. Idempotent.
5. Gold notebooks under `04 Gold/` build facts and dimensions. SCD Type 1 or Type 2 as documented per table.
6. Power BI semantic models read gold-layer tables. Refresh schedule: [schedule].
7. Consumers ([analyst team, ops team, finance]) view reports and act.

## Features

### Sources

- **{{source 1}}** — [what it provides, ingestion mechanism (API, CSV, SFTP, Monday board), refresh cadence].
- **{{source 2}}** — [...]
- [Add more sources as scoped.]

### Layers

- **Bronze** — raw + ingestion metadata. One Delta table per source object.
- **Silver** — cleansed, conformed. Business rules applied. Deduplicated on business keys.
- **Gold** — facts and dims. SCD-aware. Optimized for Power BI consumption.

### CI/CD

<!-- IF MULTI_DEV -->
- Multi-developer pattern: each developer has a `dev/<name>` branch and personal Fabric workspace.
- PRs from `dev/<name>` to `main` translate workspace IDs and lakehouse GUIDs.
- `main` represents the Test workspace.
- Test → Prod promotion via GitHub Actions and `fabric-cli`.
<!-- END IF -->
<!-- IF NOT MULTI_DEV -->
- Single-developer pattern: work happens on `main`. The Fabric workspace tracks `main` directly.
- [Document promotion path when project graduates to multi-environment.]
<!-- END IF -->

## In scope

- Fabric medallion (Bronze → Silver → Gold) for the listed sources.
- Power BI semantic model integration via gold-layer tables.
- [Project-specific in-scope items.]

## Out of scope

[Explicit list. Listing what the project will *not* build is more important than listing what it will. Examples to refine per project:]

- Real-time streaming (this is a batch project).
- Source-system master data corrections (those happen upstream in the source system).
- Power BI report design (BI team owns it; this project owns the gold tables).
- ML model training and serving.
- [Add project-specific exclusions.]

## Success criteria

[Verifiable conditions. "Looks good" is not a criterion. Examples:]

- A scheduled pipeline run from {{primary source}} completes end-to-end with no manual intervention.
- Gold-layer fact table `[name]` matches the source system's [name] report within ±[N] rows for the prior month.
- A new developer joining the project can run a verification script in `project/scripts/` and confirm their local environment is correctly configured in under [N] minutes.
- [Add project-specific criteria.]
