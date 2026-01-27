---
name: gsd:quick
description: Execute a quick task with GSD guarantees (atomic commits, state tracking) but skip optional agents
argument-hint: ""
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - AskUserQuestion
---

# Quick Task

Executes small, ad-hoc tasks with GSD guarantees (atomic commits, STATE.md tracking) while skipping optional agents (research, plan-checker, verifier). Spawns gsd-planner in quick mode plus gsd-executor(s). Quick tasks live in `.planning/quick/` separate from planned phases and update STATE.md's "Quick Tasks Completed" table, not ROADMAP.md. Use when you know exactly what to do and the task is small enough to skip research or verification.

@.planning/STATE.md

Read model profile from .planning/config.json, defaulting to "balanced" if unset.

| Agent | quality | balanced | budget |
|-------|---------|----------|--------|
| gsd-planner | opus | opus | sonnet |
| gsd-executor | opus | sonnet | sonnet |

Validate that .planning/ROADMAP.md exists; if not, stop with an error directing the user to run /gsd:new-project first. Quick tasks can run mid-phase -- validation only checks ROADMAP.md exists, not phase status.

Ask the user for a task description via AskUserQuestion. If empty, re-prompt. Generate a slug from the description (lowercase, hyphens only, max 40 chars). Ensure `.planning/quick/` exists, find the highest existing numbered directory, and increment to get the next sequential three-digit number (001, 002, 003...). Create the directory at `.planning/quick/{NNN}-{slug}/`.

Spawn gsd-planner with quick mode context: pass the directory path, description, and STATE.md reference. Constrain it to a single plan with 1-3 focused tasks, no research or checker phase, targeting ~30% context usage. The planner writes to `{QUICK_DIR}/{NNN}-PLAN.md`. Verify the plan file exists after the planner returns; error if missing.

Spawn gsd-executor with the plan reference and STATE.md. The executor implements all tasks with atomic commits and writes `{QUICK_DIR}/{NNN}-SUMMARY.md`. It must not update ROADMAP.md. Verify the summary file exists after return; error if missing. For the rare case of multiple plans, spawn executors in parallel waves per execute-phase patterns.

Update STATE.md: if a "Quick Tasks Completed" section does not exist, create it after "Blockers/Concerns" with a table (columns: #, Description, Date, Commit, Directory). Append a new row with the task number, description, date, commit hash, and a relative link to the quick task directory. Update the "Last activity" line to reflect the completed quick task. Use the Edit tool for atomic changes.

Stage the plan, summary, and STATE.md, then commit with format `docs(quick-{NNN}): {DESCRIPTION}`. Report the final commit hash, summary path, and suggest `/gsd:quick` for the next task.
