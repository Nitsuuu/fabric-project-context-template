# Progress Tracker — {{PROJECT_NAME}}

Update this file after every meaningful change. Keep entries short — this is a state log, not a journal.

## Current Phase

Initial scaffold complete. Ready to define and execute the first REQ.

## Current Goal

[Describe the immediate goal: usually the first feature or first source-system ingestion. Replace this placeholder.]

## Completed

- {{SCAFFOLD_DATE}} — Project scaffolded from `fabric-project-context-template` {{TEMPLATE_VERSION}}.
- {{SCAFFOLD_DATE}} — Six context files written (this file plus the other five in `project/context/`).
- {{SCAFFOLD_DATE}} — Medallion structure created under `workspace-assets/00 Notebooks/`.
- {{SCAFFOLD_DATE}} — REQ queue created at `project/requests/`.

## In Progress

- None yet.

## Next Up

[List the first 3-5 REQs in build order. Each REQ should be one focused unit. Examples to refine:]

1. **REQ-001 — Set up Key Vault and Variable Library wiring.** Connect Fabric workspace identity to Azure Key Vault. Configure Variable Library with environment-specific values.
2. **REQ-002 — Implement [primary source] bronze ingestion for [entity].** First end-to-end pipeline. Lands one Delta table with `etl_*` metadata.
3. **REQ-003 — Build the logging framework and control table.** Standard utility used by every subsequent pipeline.
4. **REQ-004 — Set up GitHub Actions workflow for workspace sync.** Sync `main` to the Test Fabric workspace on push.
5. [Continue per project plan.]

## Open Questions

[Unresolved decisions blocking work. Examples:]

- [Source 1] credential rotation policy — who owns it?
- Power BI semantic model refresh schedule — sync vs scheduled?
- [Source 2] historical backfill — how far back, and on what cadence?

## Architecture Decisions

[Record decisions that shape the system. Lead each entry with the date.]

- {{SCAFFOLD_DATE}} — Project scaffolded from `fabric-project-context-template` {{TEMPLATE_VERSION}}.
<!-- IF MULTI_DEV -->
- {{SCAFFOLD_DATE}} — Multi-developer pattern enabled. Branch convention: `dev/<name>` + `main`. Test workspace tracks `main`. Promotion to Prod via GitHub Actions.
<!-- END IF -->
<!-- IF NOT MULTI_DEV -->
- {{SCAFFOLD_DATE}} — Solo-developer pattern. Work happens on `main`. Multi-dev can be enabled later by manually adding the branch convention; the template's multi-dev section in `architecture.md` documents the pattern.
<!-- END IF -->

## Session Notes

- {{SCAFFOLD_DATE}} — Initial scaffold. The `/init-fabric-project` skill ran the Part 1 planning conversation and generated tailored content into `project/context/*.md`. Review each file and refine where the planning conversation surfaced project-specific detail.
