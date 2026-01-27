---
name: gsd:list-phase-assumptions
description: Surface Claude's assumptions about a phase approach before planning
argument-hint: "[phase]"
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
---

# List Phase Assumptions

Surfaces Claude's assumptions about a phase before planning begins — enabling early course correction when assumptions are wrong. Output is conversational only, no files created.

@~/.claude/get-shit-done/workflows/list-phase-assumptions.md

Load @.planning/STATE.md and @.planning/ROADMAP.md. Validate the phase number from $ARGUMENTS — error if missing, invalid, or absent from the roadmap. Analyze the roadmap description and present assumptions across five areas: technical approach, implementation order, scope boundaries, risk areas, and dependencies. End with "What do you think?" to gather feedback, then offer next steps: discuss context, plan phase, or correct assumptions.
