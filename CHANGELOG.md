# Changelog

All notable changes to `fabric-project-context-template` are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning 2.0.0](https://semver.org/).

## [Unreleased]

_No changes yet._

## [1.0.0] — 2026-05-04

Initial release. Establishes the template's structure, the scaffolding skill, and the conventions for future versions.

### Added

- **Self-describing `context/`** at the repo root. Six files (`project-overview.md`, `architecture.md`, `code-standards.md`, `ai-workflow-rules.md`, `data-conventions.md`, `progress-tracker.md`) describing the template-as-a-product.
- **Root `CLAUDE.md`** entry point declaring authoring vs. scaffolding modes.
- **Fabric-adapted blank context templates** at `templates/project/context/` — six files with `{{PLACEHOLDERS}}` and `<!-- IF FOO -->` conditional blocks.
- **`templates/CLAUDE.md`** — entry-point template for scaffolded projects.
- **`templates/.gitignore`** — tuned for Fabric, Python, secrets, and verification-script output.
- **`templates/project/requests/REQ-template.md`** — five-section spec (Goal, Design, Implementation, Dependencies, Verify when done).
- **`templates/project/requests/{queue,working,done,blocked}/`** — REQ task queue state directories.
- **`templates/workspace-assets/00 Notebooks/{01 Landing, 02 Bronze, 03 Silver, 04 Gold, 05 Utility}/`** — medallion layer scaffolds.
- **`templates/workspace-assets/02 Pipelines/`** — Fabric Pipeline scaffold.
- **`templates/.github/workflows/workspace-sync.yml`** — main → Test workspace sync starter.
- **`templates/.github/workflows/promote-to-prod.yml`** — manual main → Prod with `PROMOTE` confirmation token and GitHub Environment protection.
- **`/init-fabric-project` Claude Code skill** at `.claude/skills/init-fabric-project/`. Nine-step procedure: detect mode → identity questions → planning conversation → substitute placeholders and conditionals → write `.template-version` → inject planning content → clean up `templates/` → initial commit → next steps.
- **Skill references**: `planning-questions.md` (the seven Part 1 questions) and `placeholders.md` (substitution table + conditional-block syntax).
- **README.md** documenting the template, its scope, the upgrade story, and how to contribute patterns upstream.
- **MIT LICENSE**.

### Scope decisions

- Fabric-only. Non-Fabric platforms (Databricks, Snowflake, Synapse, BigQuery) get their own template repos.
- Batch medallion. No real-time streaming, no ML/AI pipelines.
- Claude Code only. No `AGENTS.md` / `.cursorrules` in v1.
- Full Fabric CI/CD via GitHub Actions + `fabric-cli`. The Fabric GUI Deployment Pipeline is explicitly out of scope.
- Source-system-specific bronze patterns deferred to v2 once usage shows what genuinely repeats.

[Unreleased]: https://github.com/Nitsuuu/fabric-project-context-template/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/Nitsuuu/fabric-project-context-template/releases/tag/v1.0.0
