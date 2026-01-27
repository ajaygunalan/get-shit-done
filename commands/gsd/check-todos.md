---
name: gsd:check-todos
description: List pending todos and select one to work on
argument-hint: [area filter]
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - AskUserQuestion
---

# Check Todos

Lists pending todos and routes the selected one to action. Read @.planning/STATE.md and @.planning/ROADMAP.md for context.

Count files in `.planning/todos/pending/`. If zero, tell the user no pending todos exist (captured via /gsd:add-todo) and suggest /gsd:progress or /gsd:add-todo, then exit.

If the user passed an area filter (e.g., `/gsd:check-todos api`), restrict results to that area. Parse each pending todo file for created date, title, and area. Display as a numbered list with relative ages and prompt the user to pick one by number.

On valid selection, read the full todo file and display its title, area, created date, problem, and solution sections. If it lists files, read and briefly summarize each. Then check ROADMAP.md for a phase whose area or file scope overlaps the todo.

If the todo maps to a roadmap phase, use AskUserQuestion offering: "Work on it now" (move to done, begin work), "Add to phase plan" (note reference in phase planning, keep pending), "Brainstorm approach" (keep pending, discuss), or "Put it back" (return to list). If no roadmap match, offer instead: "Work on it now", "Create a phase" (suggest /gsd:add-phase with the todo's scope, keep pending), "Brainstorm approach", or "Put it back".

For "Work on it now", move the file from `.planning/todos/pending/` to `.planning/todos/done/` and present the problem/solution context to begin work. For "Add to phase plan", note the todo reference and return to list or exit. For "Create a phase", display the /gsd:add-phase command for the user to run. For "Brainstorm approach", start discussing the problem. For "Put it back", return to the todo list.

After any action that changes the todo count, recount pending todos and update the "### Pending Todos" section in STATE.md. If a todo was moved to done/, check `.planning/config.json` for `commit_docs` (default true) and whether `.planning` is gitignored. If commits are enabled, stage the move and STATE.md changes, then commit as `docs: start work on todo - [title]`.
