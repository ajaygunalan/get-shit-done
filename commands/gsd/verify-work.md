---
name: gsd:verify-work
description: Validate built features through conversational UAT
argument-hint: "[phase number, e.g., '4']"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Edit
  - Write
  - Task
---

# Verify Work

Validates built features through conversational user acceptance testing, one test at a time. Follow @~/.claude/get-shit-done/workflows/verify-work.md using @~/.claude/get-shit-done/templates/UAT.md for output structure. Read @.planning/STATE.md and @.planning/ROADMAP.md for context.

If a phase number is provided as $ARGUMENTS, test that phase. Otherwise check for active UAT sessions to resume, or prompt for a phase number.

Check for active UAT sessions first (resume if found). Find SUMMARY.md files for the target phase and extract testable, user-observable deliverables. Create `{phase}-UAT.md` with the test list. Present tests one at a time, showing expected behavior and waiting for plain text responses — "yes/y/next" means pass, anything else is treated as an issue with severity inferred from the description. Never use AskUserQuestion for test responses, never ask severity explicitly, never show the full checklist upfront. Update UAT.md after each response. Do not fix issues during testing — log them as gaps.

On completion, commit the UAT results. Route based on outcome: if all tests pass and more phases remain, suggest /gsd:discuss-phase for the next phase. If all tests pass and this was the last phase, suggest /gsd:audit-milestone. If issues were found, spawn parallel debug agents (via Task tool) to diagnose root causes, then spawn gsd-planner in --gaps mode to create fix plans, then spawn gsd-plan-checker to verify those plans (iterate planner/checker up to 3 times). Once fix plans are verified, suggest `/gsd:execute-phase {phase} --gaps-only`. If planning is blocked after max iterations, report unresolved issues and suggest /gsd:plan-phase with --gaps for a retry or /gsd:discuss-phase to gather more context.
