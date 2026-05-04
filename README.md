# fabric-project-context-template

A GitHub template plus a Claude Code skill that scaffolds new Microsoft Fabric data engineering projects with a standardized six-file context system, a medallion lakehouse skeleton, and Fabric-native CI/CD starters.

The goal: cut the time from "new Fabric project" to "first commit on the medallion" from days to under one hour, and standardize layout, naming, and CI/CD patterns across every Fabric project.

## What you get

When you scaffold a new project from this template, you get:

```
your-new-project/
├── CLAUDE.md                              # Claude Code entry point
├── .gitignore                             # Tuned for Fabric + Python + secrets
├── .github/workflows/                     # CI/CD starters
│   ├── workspace-sync.yml                 # main → Test workspace sync
│   └── promote-to-prod.yml                # manual main → Prod with confirmation
├── workspace-assets/                      # Fabric-synced
│   ├── 00 Notebooks/{01-05 layers}/       # Landing/Bronze/Silver/Gold/Utility
│   └── 02 Pipelines/
└── project/                               # Local-only
    ├── context/                           # Six-file context system
    │   ├── project-overview.md
    │   ├── architecture.md
    │   ├── code-standards.md
    │   ├── ai-workflow-rules.md
    │   ├── data-conventions.md
    │   └── progress-tracker.md
    └── requests/                          # REQ task queue
        ├── REQ-template.md                # Five-section spec
        ├── queue/
        ├── working/
        ├── done/
        └── blocked/
```

Optional toggles add `project/docs/{guides,api,diagrams,implementations}/` and `project/scripts/`.

## How to use it

### Step 1 — Create your repo from the template

Click **Use this template** at the top of [github.com/Nitsuuu/fabric-project-context-template](https://github.com/Nitsuuu/fabric-project-context-template) and create a new repo.

### Step 2 — Clone it locally

```bash
git clone https://github.com/<your-org>/<your-new-project>.git
cd <your-new-project>
```

### Step 3 — Open in Claude Code and run the skill

```
/init-fabric-project
```

The skill will:

1. Ask 10 identity and structure questions (project name, lakehouse prefix, environments, multi-developer pattern, optional toggles).
2. Run the Part 1 planning conversation — seven questions designed to make you think clearly about what you're building before any code is written.
3. Substitute placeholders, process conditional blocks, and inject your planning-conversation answers into the six context files.
4. Record the template version in `project/context/.template-version`.
5. Delete the `templates/` directory.
6. Create the initial commit (does not push — you review and push manually).

The skill runs once per project. Re-running it on an already-scaffolded project is refused automatically.

## Scope

### In scope

- Microsoft Fabric only (Notebooks, OneLake, Delta Lake, Pipelines, Variable Library, Key Vault).
- Batch medallion architecture (Bronze/Silver/Gold). Default, but flexible — non-medallion variants are not actively prevented.
- Power BI / semantic model patterns where they touch the gold layer.
- Data quality framework hooks.
- Full Fabric CI/CD via GitHub Actions and `fabric-cli` (replacing the GUI Deployment Pipeline).
- Solo-developer projects by default; multi-developer (`dev/<name>` + GUID translation) opt-in.
- Claude Code as the coding agent.

### Out of scope

- Non-Fabric platforms (Databricks, Snowflake, Synapse, BigQuery). Other platforms get their own template repos.
- Real-time streaming (Eventstreams, Eventhouse, KQL).
- ML/AI pipelines.
- Source-system-specific bronze patterns (deferred to v2 once usage shows what genuinely repeats).
- Pipeline JSON authoring scaffolds (referenced in `architecture.md`, not generated).
- Coding agents other than Claude Code (no `AGENTS.md` / `.cursorrules`).
- External orchestrators (Airflow, ADF, Prefect).
- The Fabric GUI Deployment Pipeline feature.

## Versioning and upgrades

The template follows [Semantic Versioning 2.0.0](https://semver.org/) strictly.

- **Major (`v2.0.0`)** — breaking change to scaffolded projects.
- **Minor (`v1.x.0`)** — additive (new optional placeholder, new toggle, new pattern reference).
- **Patch (`v1.0.x`)** — typo fixes, doc clarifications.

Each release has a [CHANGELOG.md](CHANGELOG.md) entry, a git tag, and a GitHub release.

Scaffolded projects record the template version they came from in `project/context/.template-version`.

### Upgrading an existing project

Pattern propagation is **manual** by design. Auto-syncing opinionated content into projects with custom edits is risky and often clobbers important work.

To pull updates from a newer template version into your project:

1. Read the diff in [CHANGELOG.md](CHANGELOG.md) between your `.template-version` and the latest tag.
2. Decide which changes apply to your project.
3. Apply them by hand (or with a one-shot Claude Code session — the skill does not auto-upgrade).
4. Update `project/context/.template-version` to the new version.

## Contributing patterns back upstream

If a downstream project (yours, mine, anyone's) discovers a Fabric-specific pattern worth standardizing — open a PR.

Good upstream candidates:

- A new placeholder that two or more projects would benefit from.
- A CI/CD workflow improvement (better auth, better error handling, faster sync).
- A `data-conventions.md` rule discovered the hard way (e.g., the StorEdge `entity_level` flexibility, the FLOAT-to-INT composite-key gotcha).
- A new toggle for a feature that some projects need and others don't.

Not good upstream candidates:

- Source-system-specific notebooks. Those belong in a sibling skill (e.g., `cvc-do-work`), not the template.
- Project-specific naming conventions that the rest of the world won't share.
- Anything that adds a non-Fabric platform.

## Self-description

This repo describes itself using its own methodology. See `context/` for the six files that document what the template is, how it's structured, the rules it enforces, and the current build state.

## License

MIT. See [LICENSE](LICENSE).
