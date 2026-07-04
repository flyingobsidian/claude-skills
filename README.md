# Claude Skills

## Patterns

The `/patterns` skill identifies patterns and anti-patterns in a codebase, using research from the internet. The output is a list of recommendations (as Github/Gitlab Issues, or a markdown file) that can scheduled to be be worked on as developers see fit.

Repos with more than one language/framework (e.g. Terragrunt, Terraform, and one or more application languages) are supported: each such component gets its own `patterns-<name>.md` file at the repo root, e.g. `patterns-terraform.md`, `patterns-python.md`. A legacy single `patterns.md` from an older version of this skill is still recognised.

### Subskills

- `/patterns help`: list subskills.
- `/patterns research-create [<language_or_framework>...]`: Research patterns and create a new `patterns-<name>.md`. Run with no arguments to auto-detect every language/framework component in the repo (confirming the list with you) and create one patterns file per component.
- `/patterns research-update [<component>...]`: Research patterns and update existing `patterns-*.md` file(s). Run with no arguments to update every patterns file found; pass component names to update only matching files.
- `/patterns analyse`: Check if the codebase needs improvements to better follow patterns and avoid anti-patterns, across all `patterns-*.md` files. This subskill:
  - creates Github/Gitlab Issues, or
  - writes `patterns-recommendations.md` in the repo root.

## Terminology

The `/terminology` skill reads a codebase, finds terminology, creates or updates definitions, and saves the results to `TERMINOLOGY.md` in the repo root.
