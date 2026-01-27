---
name: gsd:add-phase
description: Add phase to end of current milestone in roadmap
argument-hint: <description>
allowed-tools:
  - Read
  - Write
  - Bash
---

# Add Phase

Appends a new integer phase to the end of the current milestone. Read @.planning/ROADMAP.md and @.planning/STATE.md for context.

All arguments form the phase description. If none provided, error with usage: `/gsd:add-phase <description>`.

Load `.planning/ROADMAP.md` (error if missing). Find the "## Current Milestone:" section, identify all phases under it, and extract phase numbers. Filter to integer phases only (ignore decimals like 4.1), find the maximum, and add 1 to get the next phase number. Format as two-digit.

Generate a kebab-case slug from the description. Create the phase directory at `.planning/phases/{NN}-{slug}/`.

Insert the new phase entry into ROADMAP.md after the last phase in the current milestone (before the separator or next milestone heading). Use the heading `### Phase {N}: {Description}` with Goal "[To be planned]", Depends on the previous phase, 0 plans, and a TBD plan entry referencing `/gsd:plan-phase {N}`. Preserve all other roadmap content exactly.

Update STATE.md: set the "**Next Phase:**" reference under "## Current Position", and add a "Phase {N} added: {description}" entry under "## Accumulated Context" > "### Roadmap Evolution" (create the section if absent).

Present a summary showing the phase number, description, directory path, and status. Suggest `/gsd:plan-phase {N}` as the next step (with /clear first for a fresh context window), and offer `/gsd:add-phase` to add another.
