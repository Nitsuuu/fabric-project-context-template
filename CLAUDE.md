# fabric-project-context-template — Entry Point

You are working inside `fabric-project-context-template`, a meta-project: a GitHub template repo plus a Claude Code skill that scaffolds new Microsoft Fabric data engineering projects with a six-file context system.

## Read these in order before making any change

1. `context/project-overview.md` — what this template is, who uses it, scope
2. `context/architecture.md` — repo structure, system boundaries, invariants, versioning model
3. `context/data-conventions.md` — naming and structural rules the template enforces in scaffolded projects
4. `context/code-standards.md` — conventions for authoring template content (placeholder syntax, file naming)
5. `context/ai-workflow-rules.md` — how Claude behaves when editing this repo and when running `/init-fabric-project` in a scaffolded project
6. `context/progress-tracker.md` — current phase, completed work, in progress, next up

## Two operating modes

This repo runs in two distinct modes. Confirm which one applies before acting.

- **Authoring mode** — editing the template itself (templates, skill, docs, CI/CD starters). Files in `templates/` are *templates*; do not treat their placeholders (`{{PROJECT_NAME}}`, etc.) as bugs.
- **Scaffolding mode** — running inside a *new* project that was created from this template. The `/init-fabric-project` skill executes in this mode and replaces placeholders with project-specific values.

## Update rules

- Update `context/progress-tracker.md` after every meaningful change.
- If a change affects scope, architecture, or conventions, update the relevant `context/*.md` file in the same commit.
- Never bump the SemVer tag without a corresponding `CHANGELOG.md` entry.
- Never include real workspace IDs, lakehouse GUIDs, or secrets in any file under `templates/`.
