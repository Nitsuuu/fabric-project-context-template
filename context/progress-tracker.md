# Progress Tracker

Update this file after every meaningful change. Keep entries short — this is a state log, not a journal.

## Current Phase

v1.0 development — initial template build.

## Current Goal

Ship `v1.0.0`: GitHub template repo with self-describing context files, Fabric-adapted blank templates under `templates/project/context/`, working `/init-fabric-project` skill, README, and CHANGELOG.

## Completed

- 2026-05-04 — Planning conversation (Q1-Q7) complete. All seven foundational decisions locked.
- 2026-05-04 — GitHub repo created at `Nitsuuu/fabric-project-context-template` (public, MIT).
- 2026-05-04 — Repo cloned to `/Users/blanc/Documents/10 Code/05 fabric-project-context-template/`.
- 2026-05-04 — Root `CLAUDE.md` entry point written.
- 2026-05-04 — Six self-describing context files written: project-overview, architecture, code-standards, ai-workflow-rules, data-conventions, progress-tracker (this file). Template now describes itself using its own methodology.

## In Progress

- None.

## Next Up

1. **Write Fabric-adapted blank templates** under `templates/project/context/` — six markdown files with placeholders that the skill fills in.
2. **Scaffold `templates/CLAUDE.md`** — entry-point template for scaffolded projects (with placeholders).
3. **Scaffold `templates/workspace-assets/` skeleton** — empty layer folders, lakehouse `.platform` skeletons.
4. **Scaffold `templates/project/requests/` queue** — empty `queue/working/done/blocked/` folders plus REQ-template.md (five-section structure).
5. **Scaffold `templates/.github/workflows/`** — starter Fabric CI/CD workflows (sync, test promotion, prod promotion, scheduled sync).
6. **Build `/init-fabric-project` skill** under `skill/init-fabric-project/`. SKILL.md + helper scripts + references.
7. **Replace auto-generated `README.md`** with proper template README (how to use, what gets scaffolded, version compatibility).
8. **Initialize `CHANGELOG.md`** in Keep a Changelog format.
9. **Tag `v1.0.0`** and create GitHub release.

## Open Questions

- **Skill packaging**: should `/init-fabric-project` ship as a Claude Code skill via the `.claude/` directory inside the template, or live in `claude-code-shared-config` and be installed separately? Decision affects whether the skill is automatically available in scaffolded projects on first clone, or whether the developer must sync config first.
- **First downstream test**: which existing project (CVC, UPA, or a fresh test project) is the first one to be retrofitted/scaffolded from `v1.0.0`? Retrofitting a real project surfaces gaps that a fresh scaffold won't.
- **`requirements.txt` / `pyproject.toml`**: do scaffolded projects ship a Python dependency manifest? Fabric notebooks have their own runtime, but verification scripts in `project/scripts/` may need local deps.

## Architecture Decisions

Recorded in chronological order. These are the decisions that shaped v1.

- **2026-05-04 / Q1** — Delivery model: scaffolding tool, not static skeleton or live dependency. Reason: file-tree is mechanical (handled by GitHub template); the *thinking* part needs a Claude Code skill.
- **2026-05-04 / Q2** — Scaffold scope: option (b) with toggles. Mandatory: `project/context/`, `project/requests/`, `workspace-assets/` medallion tree. Toggles: `project/docs/` subdirs, `project/scripts/`. Renames: `dev/` → `project/`, `workspace/` → `workspace-assets/`. No root-level numbering.
- **2026-05-04 / Q3** — Tech: hybrid. GitHub template (mechanical) + `/init-fabric-project` Claude Code skill (conversational). Neither alone is sufficient.
- **2026-05-04 / Q4** — Skill behavior: identity + structure questions PLUS full Part 1 planning conversation. Source-system-specific scaffolding (StorEdge, RentManager bronze patterns) deferred to v2.
- **2026-05-04 / Q5** — Branch model: solo-dev default, multi-dev (`dev/<name>`) opt-in via skill question.
- **2026-05-04 / Q6** — Out of scope: non-Fabric platforms, real-time streaming, ML/AI, source-system bronze (v2), Pipeline JSON, non-Claude agents, external orchestrators, GUI Fabric Deployment Pipelines. In scope: Fabric medallion (flexible), Power BI/semantic models, DQ frameworks, control tables + logging, Key Vault, Variable Library, **full Fabric CI/CD via GitHub Actions + fabric-cli**, verification scripts, DBML data dictionary.
- **2026-05-04 / Q7** — Pattern propagation: manual upstream + manual pull. SemVer tags + CHANGELOG. `.template-version` recorded in each scaffolded project. No auto-sync.
- **2026-05-04** — File list: kept the methodology's six-file structure but renamed `ui-context.md` → `data-conventions.md`. UI is irrelevant to Fabric notebooks; data conventions are the structural-standards equivalent.
- **2026-05-04** — Filesystem location: `/Users/blanc/Documents/10 Code/05 fabric-project-context-template/`. Slot 05 was open; flat numbered convention matches existing `10 Code/` siblings.

## Session Notes

- The planning conversation followed Part 1 of the JavaScript Mastery "Six-File Context Methodology" download (`/Users/blanc/Downloads/Six-File+Context+Methodology/`), adapted for Fabric data engineering.
- Original methodology assumes web/app development (TypeScript, React, color tokens). The Fabric adaptation drops UI-centric concepts and substitutes data-engineering equivalents.
- The template repo dogfoods the methodology — these six context files describe the template-as-a-product. If they go stale, the template is lying about itself, which is a defect to fix in the next commit.
