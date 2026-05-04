# Code Standards (for authoring this template)

These are conventions for writing content *inside this repo*. The conventions enforced in *scaffolded projects* live in `data-conventions.md`.

## Markdown style

- ATX-style headings (`#`, `##`, `###`). No setext underlines.
- One sentence per line is fine but not required. Prefer wrapping for readability in editors that don't soft-wrap.
- Fenced code blocks with explicit language tags (` ```python `, ` ```bash `, ` ```yaml `).
- Tables for any list of three or more parallel items with the same shape.
- No trailing whitespace, single blank line between sections.

## Tone

- Direct, technical, factual. State what is true and what to do.
- No marketing language ("powerful", "robust", "blazing-fast", "state-of-the-art"). State the capability, not its emotional weight.
- No emojis in template content. Emojis are acceptable in `README.md` only if they convey navigation (e.g., section markers), and only with deliberation.
- No "we" or "you" in invariants and rules — write as imperatives. ("Do not commit secrets" not "you should not commit secrets").

## File naming

- Markdown files: lowercase-kebab-case (`project-overview.md`, `data-conventions.md`).
- Skill files: SKILL.md (uppercase) per Claude Code convention; supporting files lowercase-kebab-case.
- GitHub Actions workflows: lowercase-kebab-case ending in `.yml` (not `.yaml`).
- Template files that contain placeholders: same name as the file they will become; do not add `.template` suffix. The fact that `templates/project/context/project-overview.md` is a template is conveyed by its location, not its name.

## Placeholder syntax (for templated content)

The `/init-fabric-project` skill substitutes placeholders when scaffolding. Use these tokens:

| Placeholder | What the skill substitutes | Example |
|---|---|---|
| `{{PROJECT_NAME}}` | Project name (kebab-case) | `compass-fabric-migration` |
| `{{PROJECT_DESCRIPTION}}` | One-line description | `Fabric medallion for Compass property data` |
| `{{GITHUB_OWNER}}` | GitHub user or org | `Nitsuuu` |
| `{{LAKEHOUSE_PREFIX}}` | Prefix for lakehouse names | `lh` (default) |
| `{{LAYER_LIST}}` | Active medallion layers | `Landing, Bronze, Silver, Gold` |
| `{{INCLUDE_LANDING}}` | Boolean | `true` or `false` |
| `{{ENVIRONMENTS}}` | Env list | `Dev, Test, Prod` |
| `{{MULTI_DEV}}` | Boolean | `true` or `false` |
| `{{PRIMARY_DEVELOPER}}` | Developer name | `rhenz` |
| `{{TEMPLATE_VERSION}}` | Version this project was scaffolded from | `v1.0.0` |
| `{{SCAFFOLD_DATE}}` | ISO date | `2026-05-04` |

Rules for placeholders:

- All placeholders use double curly braces and SCREAMING_SNAKE_CASE inside.
- A placeholder that is not in the table above is a defect. Either add it to the table or remove it from the template.
- When a placeholder may not have a value (toggleable feature), the surrounding template must use a conditional comment block that the skill processes:

  ```
  <!-- IF MULTI_DEV -->
  Multi-developer content here referencing dev/{{PRIMARY_DEVELOPER}}.
  <!-- END IF -->
  ```

- Conditionals can nest but should not. If a template needs more than one level of conditional, factor it into separate template files.

## When to add a placeholder

- Add a placeholder when at least two scaffolded projects would set the value differently.
- Do not add a placeholder for values that are universally constant (e.g., the medallion layer names — those stay literal).
- Do not add a placeholder for values that should never be customized (e.g., the `etl_*` metadata column convention).
- When in doubt, leave it literal. Removing a placeholder later is harder than adding one.

## Commit messages

- Imperative mood, lowercase, no period at end of subject line.
- Subject ≤ 72 characters.
- Body wraps at 72 characters and explains *why*, not *what*.
- Reference SemVer impact in commits that affect template content: `[major]`, `[minor]`, `[patch]`. The CHANGELOG curator uses these tags to assemble release notes.

Examples:

```
add data-conventions placeholder for {{LAKEHOUSE_PREFIX}} [minor]

Projects with non-default lakehouse prefixes (e.g., bronze datalake
naming standards from large enterprises) need to override "lh".
Adds the placeholder, default "lh", documented in code-standards.
```

## Versioning rules

The template follows Semantic Versioning 2.0.0 strictly.

- **Major (`v2.0.0`)** — a breaking change to scaffolded projects. Examples: renaming `project/` to something else, removing a mandatory file, changing placeholder syntax in a way that breaks existing skills.
- **Minor (`v1.1.0`)** — additive changes. New optional placeholder, new toggle, new pattern reference, new starter workflow. Existing scaffolded projects are unaffected if they ignore the addition.
- **Patch (`v1.0.1`)** — typo fixes, documentation clarifications, non-functional cleanup.

Each release requires:

1. A `CHANGELOG.md` entry under the new version heading.
2. A git tag matching the version (`git tag v1.1.0 && git push --tags`).
3. A GitHub release with the CHANGELOG entry copied into the release notes.

## Linting and validation (planned, not enforced in v1)

- `markdownlint` for markdown style.
- A custom check for unknown placeholders (`{{...}}` that don't appear in the placeholder table).
- A check that every file under `templates/project/context/` has matching section headings to ensure six-file consistency.

These are v2 concerns. v1 ships with conventions documented but not mechanically enforced.
