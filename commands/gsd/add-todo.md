---
name: gsd:add-todo
description: Capture idea or task as todo from current conversation context
argument-hint: [optional description]
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

# Add Todo

@.planning/STATE.md

Captures an idea, task, or issue that surfaces during a GSD session as a structured todo for later work. Enables "thought, capture, continue" without derailing current work.

Ensure .planning/todos/pending and .planning/todos/done directories exist. Scan existing pending todos for their area values to maintain consistency.

If $ARGUMENTS is provided, use it as the title. If not, analyze recent conversation to extract the problem, idea, or task discussed, along with relevant file paths, error messages, and constraints. Formulate a title (3-10 words, action verb preferred), a problem description, solution hints (or "TBD"), and a list of relevant files with line numbers.

Infer the area from file paths: src/api or api maps to api, src/components or src/ui to ui, src/auth to auth, src/db or database to database, tests or __tests__ to testing, docs to docs, .planning to planning, scripts or bin to tooling, and anything unclear to general. Prefer matching an existing area from the pending todos when similar.

Check for duplicates by grepping keywords from the title across .planning/todos/pending/*.md. If a potential duplicate is found, read it, compare scope, and ask the user whether to skip, replace, or add anyway.

Create the todo file at .planning/todos/pending/${date_prefix}-${slug}.md with YAML frontmatter containing created timestamp, title, area, and files, followed by Problem and Solution sections. The problem section must have enough context for a future Claude to understand weeks later.

If .planning/STATE.md exists, count pending todos and update the "Pending Todos" subsection under "Accumulated Context."

Check .planning/config.json for commit_docs (default true) and whether .planning is gitignored. If committing is enabled, stage the new todo file and STATE.md if updated, then commit with message "docs: capture todo - [title]" including the area. If commit_docs is false or .planning is gitignored, skip git operations and log that the todo was saved without committing.

Confirm by showing the file path, title, area, and file count, then offer to continue current work, add another todo, or view all todos via /gsd:check-todos.
