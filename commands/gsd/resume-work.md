---
name: gsd:resume-work
description: Resume work from previous session with full context restoration
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
  - SlashCommand
---

# Resume Work

Restores complete project context and resumes work from a previous session.

@~/.claude/get-shit-done/workflows/resume-project.md

Follow the resume-project workflow. It verifies the project exists, loads or reconstructs STATE.md, detects checkpoints (.continue-here files) and incomplete work (PLAN without SUMMARY), presents visual status, checks for CONTEXT.md before suggesting plan versus discuss, routes to the appropriate next command, and updates session continuity.
