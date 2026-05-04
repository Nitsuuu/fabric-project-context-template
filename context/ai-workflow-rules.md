# AI Workflow Rules

Direct instructions to Claude Code when working in or with this repository. These rules are imperatives, not suggestions.

## Mode detection — do this first, every session

This repo runs in two modes. Identify which mode applies before any other action.

- **Authoring mode** — the working directory is `fabric-project-context-template` itself. Files in `templates/` contain placeholders (`{{PROJECT_NAME}}`, etc.). Treat placeholders as intentional. Edit `context/`, `templates/`, `skill/`, and root files.
- **Scaffolding mode** — `/init-fabric-project` is running inside a *new* repository created from the template. Replace placeholders with real values from the planning conversation. Do not edit anything inside the original template repo.

If you are uncertain which mode you are in, ask the user before acting. The wrong mode produces broken output in either direction.

## Authoring-mode rules

1. **Read the six context files before any change.** Order: project-overview → architecture → data-conventions → code-standards → ai-workflow-rules → progress-tracker.
2. **Update `progress-tracker.md` after every meaningful change.** Same commit, not later.
3. **If a change affects scope, structure, or invariants, update `context/*.md` in the same commit.** A stale self-description is worse than no description.
4. **Never commit real secrets, GUIDs, workspace IDs, lakehouse IDs, or PATs to any file under `templates/`.** If you find one, treat it as a security incident — rotate the value and remove it.
5. **Do not introduce a new placeholder without adding it to the table in `code-standards.md`.** Unknown placeholders break the skill.
6. **Do not add support for non-Fabric platforms or non-Claude-Code agents.** Those are explicit out-of-scope items. Reject the request and reference `project-overview.md`.
7. **One unit at a time.** Do not bundle a CHANGELOG entry, a placeholder addition, a workflow update, and a doc fix into one commit. Split them so the SemVer impact of each is clear.
8. **No speculative additions.** Do not add a placeholder, toggle, or starter workflow because "we might need it." Add it when a real downstream project asks for it.

## Scaffolding-mode rules (when `/init-fabric-project` is running)

1. **Detect existing scaffold first.** If `project/context/.template-version` already exists, refuse to scaffold and offer to switch to a strictly additive update mode instead. Never overwrite without explicit user confirmation.
2. **Run identity and structure questions first**, then the Part 1 planning conversation. Do not skip the planning conversation. The point of the methodology is the conversation, not the file copy.
3. **Ask one question at a time.** Push back when an answer is vague.
4. **Do not invent values.** If the user has not provided a value for a required placeholder, ask. Do not fill in a guess.
5. **Substitute placeholders deterministically.** A placeholder is replaced everywhere it appears, with the same substituted value. Never substitute the same placeholder with different values in different files.
6. **Process conditional blocks correctly.** `<!-- IF FOO -->...<!-- END IF -->` blocks are *removed* if `FOO` is false, *kept with markers stripped* if `FOO` is true.
7. **Record the template version in `project/context/.template-version`** as the very last scaffold step, after all files are written.
8. **After scaffolding, mark the skill dormant or self-remove** so re-running it on the same project doesn't re-scaffold. Document this clearly in the skill's README.
9. **Create a single initial commit** with the scaffolded structure. Do not push automatically. Let the developer review and push.

## Spec-driven development rules (in scaffolded projects)

These rules are scaffolded into each project's own `ai-workflow-rules.md`. They are repeated here because Claude Code may be asked to *demonstrate* the workflow inside this repo.

1. Work on one REQ at a time. The active REQ moves from `project/requests/queue/` → `working/` at start, and `working/` → `done/` (or `blocked/`) at end.
2. Read the REQ's five sections (Goal, Design, Implementation, Dependencies, Verify-when-done) before writing code. If any section is unclear, ask before implementing.
3. Implement exactly what the REQ specifies. No bonus refactors, no "while we're here" cleanups, no proactive new files.
4. Update `project/context/progress-tracker.md` when starting, when blocked, and when done.
5. Run the verify-when-done checklist before declaring complete. If any item fails, the REQ stays in `working/`.

## When to ask versus when to proceed

- **Ask** when a value is missing, an instruction is ambiguous, a security boundary is involved (secrets, PATs, real GUIDs), or an irreversible action is requested (force-push, branch deletion, tag deletion, public release).
- **Proceed** when the instruction is clear, the change is reversible, the scope is bounded, and no security boundary is touched.

## Verification checklist before declaring a unit complete

- [ ] All six context files still describe reality. No stale claims.
- [ ] No new unknown placeholders introduced.
- [ ] No real secrets or GUIDs committed.
- [ ] `CHANGELOG.md` updated if the change affects scaffolded output.
- [ ] `progress-tracker.md` updated.
- [ ] If the change affects the skill, the skill still detects and refuses re-scaffold on an existing project.
- [ ] Commit message follows the convention in `code-standards.md`.
