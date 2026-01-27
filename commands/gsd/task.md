---
name: gsd:task
description: Run the full quality pipeline for a single task (discuss → plan → execute → verify) without new-project ceremony
argument-hint: "[task id (e.g. 001) OR description]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - AskUserQuestion
  - WebFetch
  - mcp__context7__*
---

# Task

Runs the full quality pipeline for a single task — discuss (capture decisions in CONTEXT.md), plan (optional research, PLAN.md creation, plan-check loop), execute (atomic commits), and verify (goal-backward verification plus conversational UAT). This is a middle ground between /gsd:quick (skips quality agents) and full milestone/project initialization. Artifacts live in `.planning/tasks/` and do not modify ROADMAP.md.

@~/.claude/get-shit-done/workflows/task-cycle.md
@~/.claude/get-shit-done/templates/context.md
@~/.claude/get-shit-done/templates/phase-prompt.md
@~/.claude/get-shit-done/templates/verification-report.md
@~/.claude/get-shit-done/templates/UAT.md

Load @.planning/PROJECT.md, @.planning/STATE.md, and @.planning/config.json if they exist. $ARGUMENTS optionally selects a task by ID or description. Follow the workflow in task-cycle.md. The pipeline creates a task directory under `.planning/tasks/`, produces CONTEXT.md (unless skipped), plan(s) with verification, SUMMARY.md per plan from execution, VERIFICATION.md, and resumable UAT.md, then updates STATE.md with a task record if it exists.
