---
name: patterns
description: |
  Research patterns and antipatterns for given languages, and apply them to a codebase. Supports repos with multiple languages/frameworks (e.g. Terraform, Terragrunt, and one or more application languages) via one `patterns-<name>.md` file per component. Use "help" to list all available subskills.
license: MIT
compatibility: any
metadata:
  author: Flying Obsidian Ltd
  version: "2.0"
allowed-tools: >-
  Bash(git:*) Bash(grep:*) Bash(find:*) Bash(gh:*) Bash(glab:*)
  Bash(mktemp:*) Bash(touch:*) Bash(cat:*) Bash(rm -f /tmp/*)
  AskFollowupQuestion Read WebSearch WebFetch Write
---

# Subskills

Parse `args` to determine which subskill to run. Use the first element of `args` as the name of the subskill to run. Pass the remainder of `args` as arguments to the subskill.

If `args` is empty or there is no matching subskill, print an error "Subskill missing/invalid", then run the **help** subskill, then stop.

---

## help subskill

Print the following to the user verbatim:

```markdown
- **help** to list subskills.
- **research-create [<language_or_framework>...]** to research patterns for one or more languages/frameworks and create a new `patterns-<name>.md`. Run with no arguments to auto-detect every language/framework component used in the repo (confirming the list with you) and create one patterns file per component.
- **research-update [<component>...]** to research and update existing `patterns-*.md` file(s). Run with no arguments to update every patterns file found; pass one or more component names to update only matching files.
- **analyse** to check if the codebase needs improvements to better follow patterns and avoid anti-patterns, across all `patterns-*.md` files found. Create Issues, or write `patterns-recommendations.md`.
```

---

## Multi-component model

A repo may contain more than one distinct technology (e.g. Terragrunt environments, Terraform modules, a Python service, a Java service). Each such **component** gets its own patterns file at the repo root, named `patterns-<slug>.md`, where `<slug>` is derived from the component's `langs` list: lowercase, join entries with dashes, replace any remaining non-alphanumeric characters with dashes. Example: `langs = ["Terraform", "Terragrunt"]` → `patterns-terraform-terragrunt.md`.

A repo created with an older version of this skill may have a single legacy `patterns.md` (no suffix) at the repo root. Treat it as just another component's patterns file: identify its `langs` from its H1 as usual. Do not rename it.

Every subskill below that needs to find "the patterns file(s)" means: every file at the repo root matching `patterns-*.md`, plus a legacy `patterns.md` if present.

---

## research-create subskill

This subskill is for creating one or more new `patterns-<slug>.md` files at the repo root, one per component.

Process:

1. If `args` is non-empty, there is exactly one component to research: its `langs` is `args`. Skip to step 3.
2. If `args` is empty, run the [component detection subprocess](#component-detection) to propose a list of components (each with a `name` and `langs`). Present the proposed list to the user and ask them to confirm, remove, merge, rename, or add components before proceeding. Use the confirmed list as the component list for the remaining steps.
3. For each component in the component list, independently perform steps 4–11:
4. Compute the target filename `patterns-<slug>.md` per the [multi-component model](#multi-component-model).
5. If a file already exists at that path:
   - Tell the user it already exists and ask whether to: overwrite it, skip this component, or update it instead (in which case, run the [research-update subskill](#research-update-subskill) process for just this one file, then move to the next component).
   - If the user chooses skip, move to the next component.
6.
   - If this is an interactive session and the user has not already been asked, ask where new patterns files should be stored. Give options:
     - Repo root (default)
     - Other directory: ask for input from the user
   - If this is a non-interactive/headless session, use the repo root.
7. Create a [temporary file](#tmpfile).
8. Create a search string by concatenating all entries in this component's `langs` space-separated, then appending "_software design patterns anti-patterns best practices_"
9. Search the internet using the search string.
10. For each of the top search results (up to a maximum of 10), follow the [webpage subprocess](#webpage-subprocess). For the first result, pass `N=0`. Record the number returned by the subprocess, and pass this updated number as `N` to the next webpage subprocess call.
11. Create in-memory deduplicated lists of patterns and anti-patterns, merging descriptions per the [deduplication](#dedup-pattern) rules and retaining all footnotes. Using these lists, along with [the template](#template), create a new `patterns-<slug>.md` in the location from step 6.
    - Update the H1 heading to replace "LANGUAGENAME" with this component's `langs`
    - Remove sample/dummy patterns, anti-patterns
    - Add a list of patterns under the H2 heading "Patterns to Follow"
    - Add a list of anti-patterns under the H2 heading "Anti-patterns to Avoid"
    - Delete the temporary file
12. Once every component has been processed, give the user a summary: files created, files skipped, and files updated in place of creation.

## research-update subskill

This subskill is for updating existing `patterns-*.md` file(s), which live at the repo root per the [multi-component model](#multi-component-model). This subskill takes optional arguments naming which components to update.

Process:

1. Find all patterns files at the repo root (see [multi-component model](#multi-component-model)). Call this list `pattern_files`. If none can be found, ask the user for their location(s). If still none can be found, then stop.
2. If `args` is non-empty, filter `pattern_files` to only those whose filename slug or H1 `langs` case-insensitively match at least one entry in `args`. If the filtered list is empty, tell the user no matching patterns files were found and stop.
3. For each file in the (filtered) list, independently perform steps 4–9:
4. Read the file and identify from its H1 heading the languages/frameworks being used. If no languages can be identified from the H1, ask the user. Call the list of languages/frameworks `langs`. Record the highest numbered footnote number, call it `N`. If no footnotes are found, set `N=0`.
5. Create a [temporary file](#tmpfile).
6. Create a search string by concatenating all entries in `langs` space-separated, then appending "_software design patterns anti-patterns best practices_"
7. Search the internet using the search string. For each of the top search results (up to a maximum of 10), follow the [webpage subprocess](#webpage-subprocess). For the first result, pass in `N` as found in step 4. Record the number returned by the subprocess, and pass this updated number as `N` to the next webpage subprocess call.
8. Create in-memory deduplicated lists of patterns and anti-patterns: Merge the original content of this file (read in step 4) with the temporary file by:
   - [deduplicating](#dedup-pattern) and summarising: Patterns to Follow
   - [deduplicating](#dedup-pattern) and summarising: Anti-patterns to Avoid
   - keeping all footnote source citations, but moving any citation definitions that are not at the end of the file to the end of the file.
9. Save the merged results back to this same file. Do not write to the temporary file. Delete the temporary file.
10. Once every file has been processed, give the user a summary of which files were updated.

## analyse subskill

This subskill reads the codebase and checks if it needs improvements to better follow patterns and avoid anti-patterns, across every component's patterns file.

Process:

1. Find all patterns files at the repo root (see [multi-component model](#multi-component-model)). Call this list `pattern_files`. If none can be found, ask the user for their location(s). If still none can be found, then stop.
2. For each file, read it and identify from its H1 heading the languages/frameworks to investigate (`langs`) and derive a component name/slug per the [multi-component model](#multi-component-model). If a file has no identifiable `langs`, warn the user and skip that file.
3. Check the repo remote once to see if [creating remote Issues](#remote-issues) is possible. Remember the state (`remote_authenticated` or `write_to_file`); this applies to every component below.
4. For each patterns file (with its own `langs` and component slug), independently perform steps 5–7:
5. Search the codebase for source files relevant to this component's `langs`. Ignore:
   - build artefacts
   - dependency directories (e.g. `node_modules`, `.dart_tool`, `build`, `target`, `__pycache__`, `.terraform`), and generated files.
6. Search the files identified in the previous step for:
   - occurrences of **anti-patterns** that are being used
   - occurrences of **patterns** that are not being followed. For each pattern, identify what code would look like if the pattern were violated, then search for that.
7. For each occurrence:
   - Create an in-memory Issue consisting of:
     - Title: a short (max 20 words) description of the problem (pattern not followed; anti-pattern being used), prefixed with this component's slug in square brackets, e.g. `[terraform] ...`, so issues from different components are distinguishable.
     - Description: a paragraph describing the problem, including:
       - the offending code snippet using triple-backticks (max 10 lines) and a language hint immediately after the opening triple-backticks
       - file reference (including full line range)
       - name and description of the pattern/anti-pattern, and which patterns file it came from
   - Check the state from step 3, and proceed as follows:
     - If `remote_authenticated`, then use a command (`gh` or `glab`) to create an Issue from the in-memory Issue.
     - If `write_to_file`, then append the in-memory Issue to `patterns-recommendations.md`.
8. Give the user a summary of the Issues created, grouped by component, including:
   - If Github/Gitlab Issues were created, then include the Issue numbers
   - Issue Titles

## <a name="component-detection"></a> Component detection subprocess

This subprocess scans the repo for recognisable project markers and proposes a list of components (each a `name` and `langs`) for confirmation by the user. It is used by `research-create` when run with no arguments.

Use `find`/`grep` to look for markers such as (this list is illustrative, not exhaustive — use judgement for markers not listed here):

| Marker | Component | langs |
|---|---|---|
| `terragrunt.hcl` files | Terragrunt | `["Terragrunt"]` |
| `*.tf` files (outside `.terraform/`) | Terraform | `["Terraform"]` |
| `pyproject.toml`, `requirements*.txt`, `setup.py`, `Pipfile` | Python | `["Python"]` |
| `pom.xml`, `build.gradle`, `build.gradle.kts` | Java | `["Java"]` |
| `pubspec.yaml` | Dart/Flutter | `["Dart", "Flutter"]` |
| `package.json` (with TypeScript config) | TypeScript | `["TypeScript"]` |
| `package.json` (no TypeScript config) | JavaScript/Node | `["JavaScript", "Node.js"]` |
| `go.mod` | Go | `["Go"]` |
| `Gemfile` | Ruby | `["Ruby"]` |
| `Cargo.toml` | Rust | `["Rust"]` |
| `*.csproj`, `*.sln` | .NET | `["C#", ".NET"]` |

Process:

1. Search the repo (ignoring dependency/build directories such as `node_modules`, `.dart_tool`, `build`, `target`, `__pycache__`, `.terraform`, `vendor`) for the markers above and any other clearly identifiable project markers.
2. Group findings into a proposed component list. Related but distinct tools (e.g. Terraform and Terragrunt) should normally be proposed as separate components unless there is reason to merge them (e.g. very few files of one kind).
3. Present the proposed component list to the user, showing for each: the component name, its `langs`, and roughly how many matching files were found. Ask the user to confirm, or to remove/merge/rename/add components.
4. Return the confirmed component list to the calling process.

## <a name="remote-issues"></a> Creating remote Issues

Use `git remote -v` to determine if the repo is Github, Gitlab, or other git repo, or not a git repo.

If the repo has a remote that is Github or Gitlab then:
1. Select a command:
   - use `gh` for Github
   - use `glab` for Gitlab
2. Check if the user is authenticated by using the `auth status` subcommand. If the user is authenticated, remember the state as `remote_authenticated`. Stop here, and return to the calling process.
3. Use the `auth login` subcommand to log in. If login succeeds, remember the state as `remote_authenticated`, stop here, and return to the calling process. If login failed, or if the user declined, continue.

If the repo does not have a remote that is Github or Gitlab, or if it was not allowed/possible to log in, then remember that creating remote Issues is not possible. Instead, be prepared to write Issues to a file `patterns-recommendations.md` in the repo root.

Check if `patterns-recommendations.md` exists in the repo root:
- If it does not exist, create it and add a H1 heading of "Pattern Recommendations".
- If it already exists, do not overwrite it. Instead, add an H2 timestamp (in `yyyy-mm-dd hh:mm` format). Issues will be appended to it.

Remember this state as `write_to_file`.

## <a name="webpage-subprocess"></a> webpage subprocess

This subprocess takes an argument `N` which is the highest known footnote number.

- Fetch and read the content of the webpage.
- Identify patterns and anti-patterns on the webpage.
- Append them all to the temporary file created at the start of the main process.
  - Patterns go in a list of patterns.
  - Anti-patterns go in a list of anti-patterns.
  - Each pattern/anti-pattern should include a footnote [citation](#citations) to the source webpage.
  - Add footnote definitions at the end of your appended block (i.e. after the patterns and anti-patterns you are adding in this subprocess call). Ensure footnote numbers start at `N+1` where `N` is the subprocess argument passed in. Keep track of the highest numbered footnote number used.
- Be verbose with the description of the patterns and anti-patterns. Do not worry about duplicates at this stage.
- Follow at most 5 links whose anchor text or surrounding context (e.g. link title, section heading) suggests they describe a named pattern or anti-pattern. For each followed link, identify and record patterns, anti-patterns along with their sources. Do not follow any links from a webpage which was not a page found directly from an internet search. Do not follow links more than one level deep from the search results.
- Return to the calling process the number that corresponds to the highest numbered footnote.

## <a name="dedup-pattern"></a> Rules for deduplicating and summarising patterns and anti-patterns

If two items share the same name, or their descriptions describe the same concept in substantially similar language, treat them as duplicates.

In the case of 2 or more duplicate items (patterns or anti-patterns), merge the items by creating a description that encompases all items without repetition. Keep all source citation footnotes.

In source citations, when the same base URL appears (with different anchors) as a source for any items in the document, collapse it to one footnote entry pointing to the base URL.

Example: `https://www.example.com/path/to/language/patterns#rule1` and `https://www.example.com/path/to/language/patterns#rule2` can be deduplicated to `https://www.example.com/path/to/language/patterns`

Multiple patterns may legitimately share the same footnote number after deduplication; this is correct.

## <a name="citations"></a> Citations

Use markdown footnotes for linking patterns and anti-patterns to their source(s). A footnote name is a unique number. When adding footnotes to an existing document, continue numbering from the highest existing footnote index.

Example:

```markdown
# Dart Patterns and Anti-patterns

## Patterns to Follow

**some-pattern**
A widget should always have a name[^1].

## Anti-patterns to Avoid

**example-anti-pattern**
A widget should never be transparent[^2].

[^1]: See [Naming Widgets](https://www.example.com/widget/names/).
[^2]: See [Widget Transparency](https://www.example.com/widget/colours/).
```

## <a name="tmpfile"></a> Temporary files

When instructed to create a temporary file, use this shell script to generate the filename and the file:

```sh
#!/usr/bin/env bash

if command -v mktemp 1>/dev/null ; then
  mktemp /tmp/patterns-XXXXXXXX.md
elif command -v date 1>/dev/null ; then
  f="/tmp/patterns-$(date '+%Y%m%d-%H%M%S').md"
  echo "$f"
  touch "$f"
else
  echo "Failed to find mktemp or date."
  exit 1
fi
```

## <a name="template"></a> Template patterns output

```markdown
# LANGUAGENAME Patterns and Anti-patterns

## Patterns to Follow

**pattern name**
Description of pattern.

## Anti-patterns to Avoid

**anti-pattern name**
Description of anti-pattern.
```
