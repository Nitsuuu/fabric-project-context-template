# Architecture

## Stack

| Layer | Technology | Role |
|---|---|---|
| Distribution | GitHub template repository | Mechanical scaffold — "Use this template" button creates a copy with identical file tree |
| Scaffolding logic | Claude Code skill (`/init-fabric-project`) | Conversational layer — asks questions, replaces placeholders, runs the Part 1 planning conversation, generates tailored content |
| Templated content | Markdown files under `templates/` with `{{PLACEHOLDER}}` syntax | The actual files that get scaffolded into new projects |
| CI/CD starters | GitHub Actions YAML under `templates/.github/workflows/` | Starter workflows for Fabric workspace sync, Test/Prod promotion via `fabric-cli` |
| Versioning | Git tags (SemVer) + `CHANGELOG.md` (Keep a Changelog format) | Release tracking; scaffolded projects record their template version in `project/context/.template-version` |

## Repository structure

```
fabric-project-context-template/
├── CLAUDE.md                              # Entry point for THIS repo (not the scaffolded project)
├── README.md                              # How to use the template
├── CHANGELOG.md                           # SemVer release notes
├── LICENSE                                # MIT
├── context/                               # Self-description (the template describing itself)
│   ├── project-overview.md
│   ├── architecture.md
│   ├── code-standards.md
│   ├── ai-workflow-rules.md
│   ├── data-conventions.md
│   └── progress-tracker.md
├── templates/                             # What gets scaffolded into new projects
│   ├── CLAUDE.md                          # Entry point template (uses {{PLACEHOLDERS}})
│   ├── workspace-assets/                  # Fabric-synced skeleton
│   │   ├── 00 Notebooks/
│   │   │   ├── 01 Landing/
│   │   │   ├── 02 Bronze/
│   │   │   ├── 03 Silver/
│   │   │   ├── 04 Gold/
│   │   │   └── 05 Utility/
│   │   ├── 02 Pipelines/
│   │   └── lh_{landing,bronze,silver,gold}.Lakehouse/  # Lakehouse skeletons
│   ├── project/                           # Local-only skeleton
│   │   ├── context/
│   │   │   ├── project-overview.md        # Fabric-adapted blank
│   │   │   ├── architecture.md
│   │   │   ├── code-standards.md
│   │   │   ├── ai-workflow-rules.md
│   │   │   ├── data-conventions.md
│   │   │   └── progress-tracker.md
│   │   ├── requests/
│   │   │   ├── queue/
│   │   │   ├── working/
│   │   │   ├── done/
│   │   │   ├── blocked/
│   │   │   └── REQ-template.md
│   │   ├── docs/                          # Optional, toggle
│   │   └── scripts/                       # Optional, toggle
│   └── .github/
│       └── workflows/                     # CI/CD starters
└── skill/
    └── init-fabric-project/               # Claude Code skill source
        ├── SKILL.md
        ├── scripts/                       # Helper scripts (placeholder substitution, etc.)
        └── references/                    # Detailed reference material loaded on demand
```

## System boundaries

- **`context/`** — describes the template itself, for contributors and Claude Code sessions working on the template. Never copied into scaffolded projects.
- **`templates/`** — content that gets copied into scaffolded projects. Files here may contain `{{PLACEHOLDERS}}` that the skill replaces. Do not put real GUIDs, secrets, or workspace IDs here.
- **`skill/`** — the `/init-fabric-project` Claude Code skill. Owns scaffolding logic, placeholder substitution, conversation flow, and the post-scaffold cleanup.
- **Root files** (`README.md`, `CHANGELOG.md`, `LICENSE`, `CLAUDE.md`) — describe the template repo as a product. Not copied to scaffolded projects.

## Storage model

- **Template content** lives as plain markdown and folder structure under `templates/`. No database, no build step.
- **Versioning** lives in git tags (SemVer) and `CHANGELOG.md`. The current version is whatever `git describe --tags --abbrev=0` returns.
- **Scaffolded project version tracking** lives in `project/context/.template-version` inside each downstream project (one line: the SemVer tag the project was scaffolded from).

## Distribution / access model

- Template repo is **public** on GitHub under `Nitsuuu/fabric-project-context-template`.
- Anyone can click **Use this template** to create a new repo.
- The Claude Code skill (`/init-fabric-project`) ships *inside* the template, so a new project gets the skill the moment it's created.
- The skill executes locally in the developer's Claude Code session. No remote service, no telemetry.
- Secrets, PATs, workspace IDs, and lakehouse GUIDs are entered by the developer at scaffold time and stored only in the developer's local repo (or wherever they choose to commit them — the template includes guidance on Key Vault and `.gitignore` patterns to keep them out of git).

## Pattern propagation model (how upstream patterns flow down)

This is critical and deliberately manual:

1. A downstream project (e.g. CVC) discovers a Fabric-specific pattern worth standardizing.
2. A contributor opens a PR against `fabric-project-context-template` updating the relevant template file or adding a new pattern reference.
3. PR is merged, a new SemVer tag is cut (`v1.x.0` for additions, `v1.0.x` for fixes, `v2.0.0` for breaking changes), and `CHANGELOG.md` is updated.
4. Downstream projects pull updates *manually* on their schedule by reading the diff between their `project/context/.template-version` and the latest tag.
5. There is no auto-sync. Auto-syncing into projects with custom edits is a v3-or-later concern.

## Invariants

The repo must never violate these rules:

1. **No real secrets, PATs, GUIDs, workspace IDs, or lakehouse IDs in any file under `templates/`.** The skill substitutes placeholders at scaffold time. If you find a real value committed under `templates/`, it is a security defect — rotate the secret and remove it.
2. **The template's own `context/` files must stay accurate.** If you change scope, structure, or invariants, update `context/*.md` in the same commit. A stale self-description means the template is lying about itself.
3. **No platform other than Microsoft Fabric.** This is a Fabric template. Patterns for Databricks, Snowflake, Synapse, BigQuery, etc., do not belong here. They belong in sibling template repos that may reference this one's architecture.
4. **No coding agent other than Claude Code in v1.** Only `CLAUDE.md` entry points. No `AGENTS.md`, `.cursorrules`, or equivalents. Multi-agent support is a v2+ decision.
5. **Mandatory scaffolded items stay mandatory.** `project/context/`, `project/requests/`, and the `workspace-assets/` medallion tree are non-negotiable in every scaffolded project. Toggles only apply to `project/docs/` and `project/scripts/`.
6. **The skill never overwrites a scaffolded file without explicit confirmation.** If `/init-fabric-project` is re-run in an already-scaffolded project, it must detect that and refuse (or operate in a strictly additive mode).
7. **Every release is tagged and changelogged.** No untagged "latest" — downstream projects need a stable reference point to record in `.template-version`.
8. **The template never bundles the downstream skills library.** `cvc-do-work`, `upa-dev-pr`, project-specific skills — none of them ship with the template. The template gets you to the *first* feature; project-specific skills are built per project as they prove useful.
