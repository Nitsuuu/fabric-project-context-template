# Placeholder Substitution Reference

Authoritative table of every placeholder the skill substitutes during scaffolding.

## Substitution rules

- All placeholders use double curly braces and SCREAMING_SNAKE_CASE: `{{PLACEHOLDER_NAME}}`.
- Substitution is deterministic: the same placeholder always becomes the same value, in every file.
- A placeholder not in this table is a defect. Either add it here (and to `code-standards.md` in the template's own context/) or remove it from the template.

## Placeholder table

| Placeholder | Source | Type | Example value |
|---|---|---|---|
| `{{PROJECT_NAME}}` | Identity Q1 | string (kebab-case) | `compass-fabric-migration` |
| `{{PROJECT_DESCRIPTION}}` | Identity Q2 | string (one line) | `Fabric medallion for Compass property data` |
| `{{GITHUB_OWNER}}` | Identity Q3 | string | `Nitsuuu` |
| `{{LAKEHOUSE_PREFIX}}` | Identity Q4 | string | `lh` |
| `{{INCLUDE_LANDING}}` | Identity Q5 | boolean | `true` or `false` |
| `{{ENVIRONMENTS}}` | Identity Q6 | string (comma-separated) | `Dev, Test, Prod` |
| `{{MULTI_DEV}}` | Identity Q7 | boolean | `true` or `false` |
| `{{PRIMARY_DEVELOPER}}` | Identity Q8 | string (lowercase) | `rhenz` |
| `{{INCLUDE_DOCS}}` | Identity Q9 | boolean | `true` or `false` |
| `{{INCLUDE_SCRIPTS}}` | Identity Q10 | boolean | `true` or `false` |
| `{{TEMPLATE_VERSION}}` | Computed (git tag at scaffold time) | string | `v1.0.0` |
| `{{SCAFFOLD_DATE}}` | Computed (today, ISO) | string | `2026-05-04` |

## Conditional blocks

The template uses HTML-comment-style conditionals to include or exclude content based on boolean placeholders.

### Syntax

```
<!-- IF FOO -->
Content shown only when FOO is true.
<!-- END IF -->
```

```
<!-- IF NOT FOO -->
Content shown only when FOO is false.
<!-- END IF -->
```

### Processing rules

- If `FOO` evaluates to `true`, keep the inner content with the `<!-- IF FOO -->` and `<!-- END IF -->` markers stripped.
- If `FOO` evaluates to `false`, remove the entire block including the markers.
- `<!-- IF NOT FOO -->` is symmetric.
- Conditionals do not nest in v1. If a template needs nested conditionals, factor it into separate template files.
- Whitespace cleanup: after removing a conditional block, collapse any resulting double-blank-lines to single blank lines so the output reads cleanly.

### Boolean evaluation

The boolean placeholders (`MULTI_DEV`, `INCLUDE_LANDING`, `INCLUDE_DOCS`, `INCLUDE_SCRIPTS`) are interpreted as:

- `true` if the user answered yes / true / y / 1.
- `false` otherwise (no / false / n / 0 / empty).

## Boolean placeholder usage in non-conditional contexts

When a boolean appears as a literal `{{MULTI_DEV}}` outside a conditional block (e.g., in a metadata file), substitute the literal string `true` or `false`.

## Files that contain placeholders

Every file under `templates/` may contain placeholders. The skill must process all of them. There is no allowlist — if a file is in `templates/`, it gets processed.

Files that intentionally do NOT contain placeholders (e.g., `.gitkeep` markers) are copied unchanged.

## Special path-based handling

- `templates/CLAUDE.md` → project-root `CLAUDE.md`.
- `templates/.gitignore` → project-root `.gitignore`.
- `templates/.github/workflows/*.yml` → project-root `.github/workflows/*.yml`.
- `templates/project/**` → project-root `project/**`.
- `templates/workspace-assets/**` → project-root `workspace-assets/**`.

## Toggle-driven path exclusions

- If `INCLUDE_LANDING` is false: skip `templates/workspace-assets/00 Notebooks/01 Landing/`.
- If `INCLUDE_DOCS` is false: do not create `project/docs/` or its subdirs.
- If `INCLUDE_DOCS` is true: create `project/docs/{guides,api,diagrams,implementations}/` even though they are not in the template (the template ships only the queue subdirs as literal directories; the docs subdirs are scaffolded by the skill on demand).
- If `INCLUDE_SCRIPTS` is false: do not create `project/scripts/`.
- If `INCLUDE_SCRIPTS` is true: create empty `project/scripts/`.

## Verification after substitution

After processing every file, scan the project root for any remaining `{{...}}` patterns. If any are found, list them and ask the user whether they are intentional (some bracketed content like `[Replace this paragraph...]` is intentional and should remain) or a defect (a missed placeholder that should have been substituted).
