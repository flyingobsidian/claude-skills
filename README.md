# Claude Skills

## Patterns

The `/patterns` skill identifies patterns and anti-patterns in a codebase, using research from the internet. The output is a list of recommendations (as Github/Gitlab Issues, or a markdown file) that can scheduled to be be worked on as developers see fit.

### Subskills

- `/patterns help`: list subskills.
- `/patterns research-create <language_or_framework>`: Research patterns and create a new `patterns.md`.
- `/patterns research-update`: Research patterns and update an existing `patterns.md`.
- `/patterns analyse`: Check if the codebase needs improvements to better follow patterns and avoid anti-patterns. This subskill:
  - creates Github/Gitlab Issues, or
  - writes `patterns-recommendations.md` in the repo root.

## Terminology

The `/terminology` skill reads a codebase, finds terminology, creates or updates definitions, and saves the results to `TERMINOLOGY.md` in the repo root.
