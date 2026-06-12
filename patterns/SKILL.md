---
name: patterns
description: |
  Research patterns and antipatterns for given languages, and apply them to a codebase. Use "help" to list all available subskills.
license: MIT
compatibility: any
metadata:
  author: Flying Obsidian Ltd
  version: "1.0"
allowed-tools: "Bash(git:*) Bash(grep:*) Bash(find:*) Bash(gh:*) Bash(glab:*) AskFollowupQuestion Read WebSearch WebFetch Write"
---

# Subskills

Parse `args` to determine which subskill to run. Use the first element of `args` as the name of the subskill to run. Pass the remainder of `args` as arguments to the subskill.

If `args` is empty or there is no matching subskill, print an error "Subskill missing/invalid", then run the **help** subskill, then stop.

---

## help subskill

Print the following to the user verbatim:

```markdown
- **help** to list subskills.
- **research-create <language_or_framework>...** to research patterns and create a new `patterns.md`.
- **research-update** to research patterns and update an existing `patterns.md`.
- **analyse** to check if the codebase needs improvements to better follow patterns and avoid anti-patterns. Create Issues, or write `patterns-recommendations.md`.
```

---

## research-create subskill

This subskill is for creating a new `patterns.md`.

Process:

1. The arguments to this subskill are names of languages/frameworks to research. Call this argument list `langs`.
2.
  - If this is an interactive session, ask the user where they want to store the file. Give options:
    - Repo root (default)
    - Other directory: ask for input from the user
  - If this is a non-interactive or headless session, set the location of the new file to the repo root.
3. Create a [temporary file](#tmpfile).
4. Create a search string by concatenating all entries in `langs` space-separated, then appending "_software design patterns anti-patterns best practices_"
5. Search the internet using the search string.
6. For each of the top search results (up to a maximum of 10), follow the [webpage subprocess](#webpage-subprocess). For the first result, pass `N=0`. Record the number returned by the subprocess, and pass this updated number as `N` to the next webpage subprocess call.
7. Create in-memory deduplicated lists of patterns and anti-patterns, merging descriptions per the [deduplication](#dedup-pattern) rules and retaining all footnotes.
8. Using lists created in the previous step, along with [the template](#template), create a new `patterns.md` in the location from step 2.
   - Update the H1 heading to replace "LANGUAGENAME" with the languages/frameworks that have been researched
   - Remove sample/dummy patterns, anti-patterns
   - Add a list of patterns under the H2 heading "Patterns to Follow"
   - Add a list of anti-patterns under the H2 heading "Anti-patterns to Avoid"
9. When finished, delete the temporary file.

## research-update subskill

This subskill is for updating `patterns.md`, which may be at the repo root, or be elsewhere. This subskill takes no arguments.

Process:

1. Search the repo for a file named `patterns.md`. If it cannot be found, ask the user for its location. If it still cannot be found, then stop.
2. Read `patterns.md` and identify from the H1 heading the languages/frameworks being used. If no languages can be identified from the H1, ask the user. Call the list of languages/frameworks `langs`. Record the highest numbered footnote number, call it `N`. If no footnotes are found, set `N=0`.
3. Create a [temporary file](#tmpfile).
4. Create a search string by concatenating all entries in `langs` space-separated, then appending "_software design patterns anti-patterns best practices_"
5. Search the internet using the search string.
6. For each of the top search results (up to a maximum of 10), follow the [webpage subprocess](#webpage-subprocess). For the first result, pass in `N` as found in step 2. Record the number returned by the subprocess, and pass this updated number as `N` to the next webpage subprocess call.
7. Create in-memory deduplicated lists of patterns and anti-patterns: Merge the original content of `patterns.md` (read in step 2) with the new temporary file by:
   - [deduplicating](#dedup-pattern) and summarising: Patterns to Follow
   - [deduplicating](#dedup-pattern) and summarising: Anti-patterns to Avoid
   - keeping all footnote source citations, but moving any citation definitions that are not at the end of the file to the end of the file.
8. Save the merged results to `patterns.md` in the same location as the file found in step 1. Do not write to the temporary file.
9. When finished, delete the temporary file.

## analyse subskill

This subskill reads the codebase and checks if it needs improvements to better follow patterns and avoid anti-patterns.

Process:

1. Search the repo for a file named `patterns.md`. If it cannot be found, ask the user for its location. If it still cannot be found, then stop.
2. Read `patterns.md`. Identify the list of languages/frameworks to investigate from the H1 heading. Call this list `langs`. If no languages/frameworks can be found, then stop.
3. Check the repo remote to see if [creating remote Issues](#remote-issues) is possible. Remember the state (`remote_authenticated` or `write_to_file`).
4. Search the codebase for source files relevant to the languages in `langs`. Ignore:
   - build artefacts
   - dependency directories (e.g. `node_modules`, `.dart_tool`, `build`, `target`, `__pycache__`), and generated files.
5. Search the files identified in the previous step for:
   - occurrences of **anti-patterns** that are being used
   - occurrences of **patterns** that are not being followed. For each pattern, identify what code would look like if the pattern were violated, then search for that.
6. For each occurrence:
   - Create an in-memory Issue consisting of:
     - Title: a short (max 20 words) description of the problem (pattern not followed; anti-pattern being used)
     - Description: a paragraph describing the problem, including:
       - the offending code snippet using triple-backticks (max 10 lines) and a language hint immediately after the opening triple-backticks
       - file reference (including full line range)
       - name and description of the pattern/anti-pattern from `patterns.md`
   - Check the state from step 3, and proceed as follows:
     - If `remote_authenticated`, then use a command (`gh` or `glab`) to create an Issue from the in-memory Issue.
     - If `write_to_file`, then append the in-memory Issue to `patterns-recommendations.md`.
7. Give the user a summary of the Issues created, including:
   - If Github/Gitlab Issues were created, then include the Issue numbers
   - Issue Titles

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
