# {{PROJECT_NAME}} — Claude Code Entry Point

> Scaffolded from `fabric-project-context-template` {{TEMPLATE_VERSION}} on {{SCAFFOLD_DATE}}.

## Read these in order before any change

1. `project/context/project-overview.md` — what this project is, who uses it, scope
2. `project/context/architecture.md` — stack, system boundaries, storage model, invariants
3. `project/context/data-conventions.md` — naming and schema rules
4. `project/context/code-standards.md` — project-specific conventions on top of `~/.claude/CLAUDE.md`
5. `project/context/ai-workflow-rules.md` — REQ workflow and behavior rules
6. `project/context/progress-tracker.md` — current phase, completed work, what's next

Update `project/context/progress-tracker.md` after every meaningful implementation change.

If a change affects scope, architecture, or conventions, update the relevant `project/context/*.md` file in the same commit.

## Critical gotchas

### Fabric Git Sync

- Only `workspace-assets/` syncs to Fabric. Files under `project/` and at the repo root are invisible to Fabric.
- Each Fabric workspace has unique lakehouse GUIDs. Do not hand-edit GUIDs.

<!-- IF MULTI_DEV -->
- Each developer has their own Fabric workspace with unique GUIDs.
- `main` represents the Test workspace.
- GUID translation is required for PRs (`dev/<name>` → `main`) and for syncs (`main` → `dev/<name>`).
- Use the project's PR and sync skills (when added). Never hand-edit GUIDs.
<!-- END IF -->

### Single source of truth

- `workspace-assets/00 Notebooks/` is canonical for all notebook code.
- Each `.Notebook/` folder's `notebook-content.py` IS the source — no parallel `.py` files exist outside.

### Three-part table names (cross-lakehouse)

Gold notebooks run in `{{LAKEHOUSE_PREFIX}}_gold` but read from `{{LAKEHOUSE_PREFIX}}_silver`. Without the lakehouse prefix, Spark looks in the current lakehouse only.

```python
# Cross-lakehouse: lakehouse.schema.table (3 parts)
SOURCE = f"{Config.SILVER_LAKEHOUSE}.{Config.SILVER_SCHEMA}.dim_asset_list"
# Same lakehouse: schema.table (2 parts)
TARGET = f"{Config.GOLD_SCHEMA}.dim_asset"
```

## Branches

<!-- IF MULTI_DEV -->
```
dev/<name>   → personal Fabric workspaces (edit + test)
main         → Test workspace (PRs land here)
             → GitHub Actions promote → Prod workspace
```

1. Claim a REQ on `main` first (prevents collisions between developers).
2. Switch to your `dev/<name>` branch; edit, commit, push.
3. PR `dev/<name>` → `main` via the project's PR skill (translates GUIDs).
4. After merge, sync `main` back into your dev branch (re-translates GUIDs).
<!-- END IF -->
<!-- IF NOT MULTI_DEV -->
```
main   → tracked by Fabric Test workspace
       → GitHub Actions promote → Prod workspace
```

Work happens on `main`. The Test workspace tracks `main` directly.
<!-- END IF -->

## Where things live

| Looking for | Path |
|---|---|
| Fabric notebooks | `workspace-assets/00 Notebooks/{01 Landing,02 Bronze,03 Silver,04 Gold,05 Utility}/` |
| Fabric pipelines | `workspace-assets/02 Pipelines/` |
| Lakehouse defs | `workspace-assets/{{LAKEHOUSE_PREFIX}}_*.Lakehouse/` |
| REQ tasks | `project/requests/{queue,working,done,blocked}/` |
| Implementation guides | `project/docs/guides/` (if scaffolded with docs toggle) |
| API docs | `project/docs/api/` (if scaffolded with docs toggle) |
| DB diagrams (DBML) | `project/docs/diagrams/` (if scaffolded with docs toggle) |
| Verification scripts | `project/scripts/` (if scaffolded with scripts toggle) |

Notebook naming: `nb_{layer}_{system}_{purpose}` (e.g. `nb_gold_storedge_orchestrator`).
