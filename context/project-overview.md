# Project Overview

## What this is

`fabric-project-context-template` is a GitHub template repository plus a Claude Code skill (`/init-fabric-project`) that scaffolds new Microsoft Fabric data engineering projects with a standardized six-file context system, a medallion lakehouse skeleton, and Fabric-native CI/CD starters. The template enforces conventions that have been proven across multiple production Fabric projects (CVC, UPA), so each new project starts from the same baseline rather than reinventing structure, naming, and deployment patterns.

## Goals

1. Cut the time from "new Fabric project" to "first commit on the medallion" from days to under one hour.
2. Standardize directory layout, naming, and CI/CD patterns across every Fabric project so a developer (or a fresh Claude Code session) can move between projects without re-learning structure.
3. Capture hard-won Fabric-specific patterns (GUID translation, Variable Library, Key Vault, control tables, logging framework) once, in one place, and propagate them to every new project automatically.
4. Pair the structural scaffold with the methodology's *planning conversation* so projects are designed before they are built, not during.
5. Stay narrow on purpose — Fabric only, batch only, Claude Code only — so the template remains sharp instead of generic.

## Core user flow

1. Developer clicks **Use this template** on the GitHub repo and creates a new repository.
2. Developer clones the new repo locally and opens it in Claude Code.
3. Developer runs `/init-fabric-project`. The skill asks identity and structure questions (project name, lakehouse names, environments, multi-dev opt-in, optional toggles), then runs the methodology's Part 1 planning conversation (what does this project do, who uses it, sources, complex parts, scope).
4. The skill writes tailored content into the six `project/context/*.md` files, scaffolds the `workspace-assets/` medallion tree, the `project/requests/` queue, and any opted-in `project/docs/` and `project/scripts/` directories.
5. The skill records the template version in `project/context/.template-version` and removes itself (or marks itself dormant) so future runs don't re-scaffold.
6. Developer commits the scaffolded structure, opens the first feature spec under `project/requests/queue/`, and starts building.

## Features

### Scaffolding (mandatory in every scaffolded project)

- Six-file context system at `project/context/` (project-overview, architecture, code-standards, ai-workflow-rules, data-conventions, progress-tracker).
- Medallion lakehouse tree at `workspace-assets/00 Notebooks/` (Landing, Bronze, Silver, Gold, Utility) and `workspace-assets/lh_*.Lakehouse/` skeletons.
- Pipelines folder at `workspace-assets/02 Pipelines/`.
- Request queue at `project/requests/{queue,working,done,blocked}/` with a five-section REQ template (Goal, Design, Implementation, Dependencies, Verify-when-done).
- Root `CLAUDE.md` entry point pointing at the six context files.
- `.gitignore` tuned for Fabric (sensible exclusions for local-only Fabric artifacts, secrets, derived files).
- `.template-version` recording the template version the project was scaffolded from.

### Scaffolding (toggleable per project)

- `project/docs/` subdirectories (`guides/`, `api/`, `diagrams/`, `implementations/`).
- `project/scripts/` for verification and diagnostic scripts.
- Multi-developer pattern (`dev/<name>` branch model + GUID translation hints in `CLAUDE.md`).
- GitHub Actions starter workflows for Fabric CI/CD (workspace sync, Test promotion, Prod promotion, scheduled syncs).

### Pattern coverage (referenced in `architecture.md` of scaffolded projects)

- Control tables and logging framework patterns.
- Key Vault / `notebookutils.credentials.getSecret` patterns.
- Variable Library and environment-specific configuration.
- Full Fabric CI/CD via GitHub Actions and `fabric-cli` (replacing the GUI Deployment Pipeline).
- DBML data dictionary patterns.
- Power BI / semantic model conventions for the gold layer downstream.
- Optional data quality framework hooks.

## In scope

- Microsoft Fabric only.
- Batch medallion architecture (Bronze/Silver/Gold) as the *default* — the template is flexible enough to accommodate non-medallion variants without fighting them.
- Power BI / semantic model conventions where they touch the gold layer.
- Data quality framework patterns.
- Full Fabric CI/CD via GitHub Actions and `fabric-cli`.
- Claude Code as the coding agent (`CLAUDE.md` entry point only).
- Solo-developer projects by default; multi-developer pattern available as a toggle.

## Out of scope

- Non-Fabric platforms (Databricks, Snowflake, Synapse, BigQuery). Other platforms get their own template repos.
- Real-time streaming (Eventstreams, Eventhouse, KQL).
- ML/AI pipelines (training, registry, inference).
- Source-system-specific bronze patterns (deferred to v2 once usage shows what genuinely repeats).
- Pipeline JSON authoring scaffolds (referenced in `architecture.md`, not generated).
- Coding agents other than Claude Code (no `AGENTS.md`, `.cursorrules`, etc., for v1).
- External orchestrators (Airflow, ADF, Prefect).
- The Fabric GUI Deployment Pipeline feature (replaced by scripted CI/CD).

## Success criteria

- A developer can go from `gh repo create --template` to a fully scaffolded, named, planning-conversation-completed project in under 30 minutes.
- Two scaffolded projects, opened side-by-side, have identical top-level structure and identical context-file section headings, even though their content differs.
- A fresh Claude Code session opening any scaffolded project reads `CLAUDE.md`, follows the six files in order, and can answer "what is this project, what is the architecture, where do new requests go" without further prompting.
- A pattern discovered in a downstream project (CVC, UPA) can be upstreamed to the template via a single PR, tagged with a SemVer bump, and a CHANGELOG entry — and downstream projects can pull it on their own schedule by reading the diff.
- The template repo describes itself using its own six-file system, and that description stays accurate as the template evolves.
