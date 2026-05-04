---
name: init-fabric-project
description: Scaffold a new Microsoft Fabric data engineering project from the fabric-project-context-template. Runs once per project. Asks identity and structure questions, runs the Part 1 planning conversation, substitutes placeholders, processes conditional blocks, and creates the initial commit. Use when the user runs /init-fabric-project, says "scaffold this", "init the template", "set up this Fabric project", or has just cloned a repo created from `fabric-project-context-template`.
---

# init-fabric-project

You are scaffolding a new Microsoft Fabric data engineering project from `fabric-project-context-template`. This skill runs ONCE per project. After it completes, the project no longer needs the skill except for upgrades.

## Step 1 — Detect mode and refuse if already scaffolded

Before doing anything, check the working directory:

- If `project/context/.template-version` exists, this project has already been scaffolded. **Refuse to scaffold.** Tell the user: "This project already has `project/context/.template-version`. Re-running the skill would overwrite the project's customized context files. If you want to upgrade to a newer template version, follow the upgrade guide in the template's README — do not re-run this skill."
- If `templates/` directory does not exist, the skill is being run outside a fresh template clone. Tell the user: "I can't find the `templates/` directory. This skill must run from the root of a repository created from `fabric-project-context-template`. Are you in the right directory?"
- Otherwise, proceed.

## Step 2 — Ask identity and structure questions

Ask one at a time. Push back on vague answers. Record each answer for use in Step 4.

1. **Project name** — kebab-case, no spaces. Example: `compass-fabric-migration`. Used for `{{PROJECT_NAME}}`.
2. **One-line description** — what does this project do? Used for `{{PROJECT_DESCRIPTION}}` and the GitHub repo description if applicable.
3. **GitHub owner** — user or organization that owns the repo. Used for `{{GITHUB_OWNER}}`.
4. **Lakehouse prefix** — default `lh`. Most projects accept the default. If the user has an organizational naming standard, use theirs. Used for `{{LAKEHOUSE_PREFIX}}`.
5. **Include landing layer?** — yes/no. If no, the `01 Landing/` directory and the `{{LAKEHOUSE_PREFIX}}_landing` lakehouse references are removed. Used for `{{INCLUDE_LANDING}}`.
6. **Environments** — comma-separated list. Default `Dev, Test, Prod`. Used for `{{ENVIRONMENTS}}`.
7. **Multi-developer project?** — yes/no. If yes, the `dev/<name>` branch convention and GUID-translation patterns are scaffolded into `CLAUDE.md` and `architecture.md`. Used for `{{MULTI_DEV}}`.
8. **Primary developer** — name in lowercase, used in branch examples. Used for `{{PRIMARY_DEVELOPER}}`.
9. **Include `project/docs/` subdirectories?** — yes/no. If yes, scaffolds `guides/`, `api/`, `diagrams/`, `implementations/`. If no, skip the directory entirely.
10. **Include `project/scripts/` directory?** — yes/no. For verification and diagnostic scripts.

After all answers, summarize them back to the user and ask for confirmation before proceeding to Step 3.

## Step 3 — Run the Part 1 planning conversation

Load `references/planning-questions.md` for the full prompt set. Ask one question at a time. The goal is to gather enough material to fill in the tailored content of `project/context/project-overview.md` (especially Goals, Core User Flow, Sources, In Scope, Out of Scope, Success Criteria) and `project/context/architecture.md` (especially the project-specific invariants).

Do not skip this step. The methodology's value is in the conversation, not the file copy.

When the conversation is complete, summarize the captured material as a structured list back to the user and ask for confirmation before proceeding to Step 4.

## Step 4 — Substitute placeholders and process conditionals

Load `references/placeholders.md` for the full placeholder table and the conditional-block syntax.

For each file under `templates/`:

1. Read the file.
2. Replace every placeholder with the captured value, deterministically (the same placeholder always replaced with the same value).
3. Process every `<!-- IF FOO -->...<!-- END IF -->` block:
   - If `FOO` is true, keep the inner content with the markers removed.
   - If `FOO` is false, remove the entire block including markers.
   - Process `<!-- IF NOT FOO -->...<!-- END IF -->` symmetrically.
4. Write the result to the corresponding path at the project root (move `templates/CLAUDE.md` → `CLAUDE.md`, `templates/project/...` → `project/...`, `templates/workspace-assets/...` → `workspace-assets/...`, `templates/.gitignore` → `.gitignore`, `templates/.github/workflows/...` → `.github/workflows/...`).

For binary or non-text files (none in v1, but future-proof), copy without modification.

For directories that should be excluded based on toggles:

- If the user said no to landing layer, skip `templates/workspace-assets/00 Notebooks/01 Landing/`.
- If the user said no to docs subdirs, skip `templates/project/docs/` (which currently does not exist as a literal directory in the template — it's created by this step only if the user opted in; for v1, simply create the four subdirs `guides/`, `api/`, `diagrams/`, `implementations/` under `project/docs/` if opted in).
- If the user said no to scripts dir, do not create `project/scripts/`.

## Step 5 — Write the version marker

Write `project/context/.template-version` with a single line: the SemVer tag of the template the project was scaffolded from. If the skill cannot determine the tag (e.g., the template was forked or pulled at a non-tag commit), fall back to the short commit SHA prefixed with `commit:`.

## Step 6 — Inject dynamic content

For files where the planning conversation produced specific content (project-overview.md goals, sources, in/out of scope, success criteria; architecture.md project-specific invariants), replace the bracketed placeholder content (`[Replace this paragraph...]`, `[Add project-specific...]`) with the captured material. Do not invent content — only insert what the user explicitly stated.

If a section was not addressed in the planning conversation, leave the bracketed placeholder in place so the user knows to come back and fill it in.

## Step 7 — Clean up

Delete the `templates/` directory at the project root. It is no longer needed.

The skill (`.claude/skills/init-fabric-project/`) stays in place. It contains the refusal logic in Step 1 that prevents re-scaffolding, and it may be useful for future template upgrades.

## Step 8 — Create the initial commit

Run:

```bash
git add -A
git status
```

Show the user what will be committed. Get explicit approval before creating the commit. Then:

```bash
git commit -m "scaffold {{PROJECT_NAME}} from fabric-project-context-template {{TEMPLATE_VERSION}}"
```

Do NOT push. The user reviews and pushes manually.

## Step 9 — Tell the user what to do next

Summarize:

- Where the six context files are.
- Where the REQ template is.
- That `project/context/progress-tracker.md` is the next file to read and update as work begins.
- That the suggested first REQs in `project/context/progress-tracker.md` are starting points, not gospel — the user should adjust them based on what they actually want to build first.
- The SemVer tag they were scaffolded from, and where to find the CHANGELOG to upgrade later.

## Rules and guarantees

- **Never overwrite an existing scaffolded project.** Step 1's refusal logic is non-negotiable.
- **Never invent values.** If the user has not provided a value for a placeholder, ask. Do not guess.
- **Substitute deterministically.** The same placeholder always becomes the same value, in every file.
- **Process conditionals correctly.** Wrong handling produces broken output that the user has to clean up by hand.
- **Do not push.** Initial commit only. The user pushes when ready.
- **No telemetry, no remote calls.** Everything happens locally in the user's Claude Code session.
