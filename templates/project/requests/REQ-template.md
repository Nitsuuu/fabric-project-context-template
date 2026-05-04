# REQ-NNN: [Feature Name]

> Status: queue / working / done / blocked
> Owner: [name]
> Created: YYYY-MM-DD
> Branch: [main, or dev/<name>]

## Goal

[One or two sentences. What does this REQ produce when complete? Be concrete and specific. Bad: "improve ingestion." Good: "Add a Bronze ingestion notebook for StorEdge `Logi_Company_Auction_Schedule` that runs once per company and writes a partitioned Delta table with `etl_*` metadata."]

## Design

[Visual and structural decisions specific to this REQ. Reference `project/context/data-conventions.md` for naming and schema rules. Describe:]

- **Layer**: [landing / bronze / silver / gold / utility]
- **Source system**: [if applicable]
- **Target table(s)**: `[lakehouse].[schema].[table]`
- **Grain**: [the unit of one row — what it represents]
- **SCD type**: [Type 1 / Type 2 / N/A]
- **Partitioning**: [partition columns]
- **Idempotency strategy**: [partition overwrite / MERGE / other]

## Implementation

[Break into sub-sections — one per component or system boundary. Each sub-section tells the agent exactly what to build.]

### [Component or notebook]

[Detailed description of what to build. Be specific. Include file paths.]

### [Next sub-section]

[Description.]

## Dependencies

[Any prerequisites this REQ needs that aren't already in place. List explicitly. Examples:]

- REQ-XXX must be complete (provides [what]).
- Key Vault secret `[name]` must exist.
- Variable Library entry `[name]` must be set for [environments].
- Source system access verified (PAT, API key, IP allowlist).

## Verify when done

[A checklist of specific conditions that must be true. The agent checks against these. The reviewer checks against these. Do not declare done until every box is checked.]

- [ ] Notebook exists at `workspace-assets/00 Notebooks/[layer]/[notebook_name].Notebook/`
- [ ] Notebook runs end-to-end without error in Fabric
- [ ] Target table exists with correct schema and `etl_*` metadata columns
- [ ] Pipeline is idempotent (run twice, same result)
- [ ] Control table receives a row at start and an updated row at end
- [ ] Logging written via `nb_utils_logging_framework`
- [ ] No `.count()` / `.collect()` inside loops
- [ ] No hardcoded environment values (Variable Library / Key Vault)
- [ ] No real secrets in any committed file
- [ ] `project/context/progress-tracker.md` updated
- [ ] Commit message follows project convention

## Notes

[Free-form. Capture decisions made during implementation, surprising behavior of the source system, things future maintainers should know. Empty at REQ creation; populated as work progresses.]
