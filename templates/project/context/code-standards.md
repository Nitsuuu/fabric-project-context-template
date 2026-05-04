# Code Standards — {{PROJECT_NAME}}

> Inherits from the global Fabric standards at `~/.claude/CLAUDE.md`. This file documents project-specific standards on top of the global rules.

## Global standards (canonical reference)

The following are enforced project-wide and live in `~/.claude/CLAUDE.md`:

- PySpark imports: `import pyspark.sql.functions as F` + `import builtins`. Never wildcard.
- Use `builtins.sum()` / `builtins.max()` for Python data; `F.sum()` / `F.max()` for DataFrame columns. Fabric shadows built-ins.
- No `.count()` in loops. Single-pass aggregations.
- Schemas defined explicitly. No `inferSchema` in production.
- Idempotent pipelines (partition overwrite, MERGE).
- `notebookutils`, not `dbutils`.

If `~/.claude/CLAUDE.md` and this file conflict, the global file wins. Document the conflict and either update the global file or remove the conflicting rule from this one.

## Project-specific standards

[Add rules unique to {{PROJECT_NAME}}. Examples to refine or remove:]

### Source system parsing

- [Source 1] reports parse with `[date format]`.
- [Source 2] requires explicit `[parameter]` to control [behavior].

### Naming overrides

- [If the project deviates from the canonical `nb_{layer}_{system}_{purpose}` convention, document the deviation here and the reason.]

### Composite key conventions

- Use `[system_prefix]` for keys originating from {{source 1}}.
- Use `[system_prefix]` for keys originating from {{source 2}}.
- Cross-system unified keys use `coalesce` over per-system hashes.

### Control table writes

- Every pipeline writes a row at start (status `running`) and updates at end.
- Use `from nb_utils.nb_utils_control_table_writer import write_control_metrics`.
- Required fields: `pipeline_name`, `source_system`, `layer`, `records_processed`, `execution_time_seconds`, `status`.

### Logger patterns

- Use the `NullLogger` pattern for class-internal logging. Classes never receive `None` for logger.
- Do not pass `exc_info=True` to `NotebookLogger`. It is not supported.

### File organization

- One executor notebook per source system per layer (Bronze, Silver, Gold).
- One framework notebook per system per layer for Silver and Gold (`nb_{layer}_{system}_main_framework`).
- One orchestrator per system per layer for Silver and Gold (`nb_{layer}_{system}_orchestrator`).

### Verification scripts

- Live in `project/scripts/`. Never deployed to Fabric.
- Naming: `verify_<feature>_<date>.py` for one-shot verification, `diagnose_<symptom>.py` for triage.
- Outputs (CSV, logs) live alongside the script in `project/scripts/<script_name>_results/` or are gitignored.

### Documentation patterns

- New patterns discovered during the build: capture in `project/docs/implementations/REQ-<id>.md`.
- API quirks: capture in `project/docs/api/<source_system>/`.
- Schema diagrams: DBML format in `project/docs/diagrams/`.

## Conflicts with the global file

[Document any deliberate deviation from `~/.claude/CLAUDE.md`. Empty by default. If this section grows, the global file may need updating.]

- None at scaffold time.
