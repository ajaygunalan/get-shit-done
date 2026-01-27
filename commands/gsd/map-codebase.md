---
name: gsd:map-codebase
description: Analyze codebase with parallel mapper agents to produce .planning/codebase/ documents
argument-hint: "[optional: specific area to map, e.g., 'api' or 'auth']"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Write
  - Task
---

# Map Codebase

Analyzes an existing codebase using parallel gsd-codebase-mapper agents that write structured documents directly to `.planning/codebase/`, keeping orchestrator context minimal. Output: 7 documents in `.planning/codebase/`. Use for brownfield projects before initialization, refreshing understanding after significant changes, onboarding to unfamiliar codebases, or before major refactoring. Skip for greenfield projects with no code or trivial codebases under 5 files.

@~/.claude/get-shit-done/workflows/map-codebase.md

Load @.planning/STATE.md if it exists. $ARGUMENTS optionally narrows the focus to a specific subsystem. If `.planning/codebase/` already exists, offer to refresh or skip. Create the directory, then spawn 4 parallel gsd-codebase-mapper agents: tech focus (writes STACK.md, INTEGRATIONS.md), arch focus (writes ARCHITECTURE.md, STRUCTURE.md), quality focus (writes CONVENTIONS.md, TESTING.md), and concerns focus (writes CONCERNS.md). Collect confirmations only — not document contents. Verify all 7 documents exist with line counts, commit the codebase map, and offer next steps (typically /gsd:new-project or /gsd:plan-phase).
