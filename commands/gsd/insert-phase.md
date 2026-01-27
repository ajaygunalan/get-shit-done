---
name: gsd:insert-phase
description: Insert urgent work as decimal phase (e.g., 72.1) between existing phases
argument-hint: <after> <description>
allowed-tools:
  - Read
  - Write
  - Bash
---

# Insert Phase

Inserts a decimal phase for urgent work discovered mid-milestone, preserving the existing phase sequence without renumbering. Read @.planning/ROADMAP.md and @.planning/STATE.md for context.

Parse arguments: first argument is the integer phase number to insert after, remaining arguments form the description. Both are required — error if missing. Validate the first argument is an integer.

Load `.planning/ROADMAP.md` (error if missing). Verify a `### Phase {N}:` heading exists for the target phase number and that it belongs to the current milestone. If not found, list available phases and exit.

Find all existing decimal phases under that integer (e.g., 72.1, 72.2) by searching for `### Phase {N}.M:` headings. Calculate the next decimal as max existing suffix + 1, or .1 if none exist.

Generate a kebab-case slug from the description. Create the phase directory at `.planning/phases/{decimal_phase}-{slug}/`.

Insert the new phase entry into ROADMAP.md immediately after the target phase's content (before the next phase heading or separator). Use the heading format `### Phase {decimal_phase}: {Description} (INSERTED)` with Goal set to "[Urgent work - to be planned]", Depends on pointing to the target phase, 0 plans, and a TBD plan entry referencing `/gsd:plan-phase {decimal_phase}`. Preserve all other roadmap content exactly.

Update STATE.md under "## Accumulated Context" > "### Roadmap Evolution" (create the section if absent) with an entry noting the phase insertion as URGENT.

Present a summary showing the decimal phase number, description, directory path, and status. Suggest `/gsd:plan-phase {decimal_phase}` as the next step (with /clear first for a fresh context window), and note the user should review whether the next integer phase's dependencies still hold.
