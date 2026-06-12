---
name: terminology
description: Read a codebase, find terminology, create/update definitions into a markdown document.
license: MIT
compatibility: any
metadata:
  author: Flying Obsidian Ltd
  version: "1.0"
allowed-tools: "Bash(git:*) Bash(grep:*) Read"
---

# Terminology

Apply this process when:
- creating or updating a `TERMINOLOGY.md` markdown document; or
- deciding which words to use when communicating to developers using chat or documentation.

## What is terminology?

Terminology is special words or expressions that are:
- used in relation to a particular subject or activity
- in need of a definition which is specific to the project being investigated

## What to include as terminology

- words and phrases that take on a _specific_ meaning in this project that overrides a _generic_ meaning that an uninformed person might otherwise assume. For example, "Deck" means "a group of flashcards on a particular topic". An uninformed person might guess that "Deck" means "Deck of 52 playing cards (Hearts, Clubs, Diamonds, Spades)" or "Deck of a ship".
- words and phrases that enable more terse communication. For example, if we all agree what "spaced repetition" is, then there is no need to repeatedly use "the process by which items to be memorised are practiced with increasing timespans between them".

The overall goal of agreeing and using terminology is to reduce the verbosity (as measured by number of tokens) in communication.

## What to exclude as terminology
- general words and phrases that would be understood by anyone who already has a sufficient understanding of the IT world: computers, programming, git. For example, any developer will understand "feature", "pull request", "debug", "test-driven development". These words and phrases do not take on any special meaning in _this_ project.

## A terminology entry

A terminology entry contains:
- the word or phrase
- the project-specific definition
- (optional) context: Is the word or phrase only used in the front-end, back-end, or with one particular language or framework? If yes, include context.

## First-time Process

Use this process if there is no existing `TERMINOLOGY.md` file in the repo root.

1. Create an empty `TERMINOLOGY.md` file in the repo root. Use [`assets/TERMINOLOGY.md`](assets/TERMINOLOGY.md) as a template. Keep the markdown header in the template. Use an unordered markdown list. Use [HTML anchors](#html-anchors) so that terminology items can be directly linked to.
2. Scan (using `git ls-files --exclude-standard` and `grep`) the codebase for:
   - source code, including tests
   - documentation
   - continuous integration (CI) files
3. Look for terminology words and phrases that are appropriate (according to the above include/exclude rules) for `TERMINOLOGY.md`. For each new word/phrase, create a succinct (generally under 20 words) definition that is specific to this project. In the case of ambiguity, question the developer.
4. Add the terms and definitions found in step 3 to `TERMINOLOGY.md` in alphabetical order.

## Subsequent Process

Use this process if `TERMINOLOGY.md` already exists in the repo root.

1. Read `TERMINOLOGY.md`. This is the list of words and phrases that will be updated, improved, corrected.
2. Scan (using `git ls-files --exclude-standard` and `grep`) the codebase for:
   - source code, including tests
   - documentation
   - continuous integration (CI) files
3. In the codebase, look for terminology words and phrases that already exist in `TERMINOLOGY.md`. Check each one to ensure the definition matches the usage in the codebase. If it does not, update the definition in `TERMINOLOGY.md`. In the case of ambiguity, question the developer.
4. Look for new words and phrases that would make appropriate (according to the above include/exclude rules) additions to `TERMINOLOGY.md`. For each new word/phrase, create a succinct (generally under 20 words) definition that is specific to this project. In the case of ambiguity, question the developer. If a term has multiple meanings, follow the Conflicts section below.
5. Add the new words and phrases to `TERMINOLOGY.md`, keeping alphabetical order.
6. Also check for obsolete terminology, i.e. words and phrases that are included in `TERMINOLOGY.md`, but no longer referred to in the codebase. Move obsolete terminology to a separate subsection at the end of the file. A developer will remove these manually. If terminology in the obsolete subsection is found not to be obsolete, move it back to the main list, keeping alphabetical order.

## Conflicts

In the case of a terminology word or phrase which has multiple meanings, depending on context, add a definition for each, one after the other. Ensure the [HTML anchor](#html-anchor) is unique. For example:

```markdown
- <a id="flibble-backend"></a> **flibble**: a database that flobbles regularly. Context: back-end database.

- <a id="flibble-frontend"></a> **flibble**: a type of widget that appears as a button on the Home screen. Context: front-end widget.
```

## <a id="html-anchor"></a> HTML anchors

For HTML anchors, IDs should be lowercase with hyphens, for example "word", "multi-word-phrase".

Only add the context to the ID if this resolves ambiguity and keeps IDs unique. For example, use "card" if unambiguous, but "card-frontend" and "card-backend" if there are multiple meanings (as shown in the Conflicts section).
