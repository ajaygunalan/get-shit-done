---
name: gsd:execute-phase
description: Execute all plans in a phase with wave-based parallelization
argument-hint: "<phase-number> [--gaps-only]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - TodoWrite
  - AskUserQuestion
---

# Execute Phase

Execute all plans in a phase using wave-based parallel execution. The orchestrator stays lean (~15% context budget): discover plans, analyze dependencies, group into waves, spawn gsd-executor subagents (each gets a fresh 100% context window), collect results.

@~/.claude/get-shit-done/references/ui-brand.md
@~/.claude/get-shit-done/workflows/execute-phase.md

Phase argument: $ARGUMENTS. The `--gaps-only` flag restricts execution to plans with `gap_closure: true` in frontmatter (used after verify-work creates fix plans).

@.planning/ROADMAP.md
@.planning/STATE.md

Read `.planning/config.json` for model_profile (default: "balanced").

| Agent | quality | balanced | budget |
|-------|---------|----------|--------|
| gsd-executor | opus | sonnet | sonnet |
| gsd-verifier | sonnet | sonnet | haiku |

Validate the phase directory exists and contains PLAN.md files -- error if none found. List all *-PLAN.md files, check which have matching *-SUMMARY.md (already complete). If `--gaps-only`, filter to plans with `gap_closure: true`. Build the list of incomplete plans. Read the `wave` field from each plan's frontmatter and group plans by wave number.

Execute waves sequentially, plans within each wave in parallel. Before spawning, read all plan file contents and STATE.md into variables -- the `@` syntax does not work across Task() boundaries. Spawn all plans in a wave with a single message containing multiple Task calls: `Task(prompt="Execute plan at {path}\n\nPlan:\n{content}\n\nProject state:\n{state}", subagent_type="gsd-executor", model="{model}")`. All run in parallel; Task blocks until all complete. No polling, no background agents, no TaskOutput loops. After each wave completes, verify SUMMARYs were created, then proceed to the next wave.

Plans with `autonomous: false` have checkpoints. The subagent pauses at the checkpoint and returns structured state; the orchestrator presents it to the user, collects the response, then spawns a fresh continuation agent. See @~/.claude/get-shit-done/workflows/execute-phase.md for complete checkpoint handling.

During execution, handle discoveries: auto-fix bugs, auto-add security/correctness gaps, auto-fix blockers -- document all in Summary. Only ask the user about major architectural/structural changes.

Commit rules: after each task, stage only files modified by that task and commit as `{type}({phase}-{plan}): {task-name}`. After all tasks in a plan, stage only PLAN.md and SUMMARY.md and commit as `docs({phase}-{plan}): complete [plan-name] plan`. Never use `git add .`, `git add -A`, or broad directory adds -- always stage files individually.

After all plans complete, check for uncommitted orchestrator corrections via `git status --porcelain` and commit them as `fix({phase}): orchestrator corrections` if any exist. Then check `.planning/config.json` for the `verifier` toggle (default: true). If enabled, spawn a gsd-verifier subagent to check must_haves against the actual codebase (not SUMMARY claims). The verifier creates VERIFICATION.md. If `passed`, continue. If `human_needed`, present items and get approval or feedback. If `gaps_found`, present gaps and offer `/gsd:plan-phase {X} --gaps`.

Update ROADMAP.md and STATE.md to reflect phase completion. Read the phase's `Requirements:` line from ROADMAP.md, then update REQUIREMENTS.md traceability table changing each listed REQ-ID status from "Pending" to "Complete" (skip if REQUIREMENTS.md doesn't exist or phase has no Requirements line).

Check `commit_docs` in config.json (default: true). If enabled, stage ROADMAP.md, STATE.md, and REQUIREMENTS.md (if updated), then commit as `docs({phase}): complete {phase-name} phase`.

Route next steps based on outcome: if `gaps_found`, suggest `/gsd:plan-phase {Z} --gaps` (after gap planning, user re-runs /gsd:execute-phase to execute new plans, verifier runs again -- loop until passed). If `passed` and more phases remain, suggest `/gsd:discuss-phase {Z+1}` with `/gsd:plan-phase {Z+1}` and `/gsd:verify-work {Z}` as alternatives. If `passed` and this was the last phase, suggest `/gsd:audit-milestone` with `/gsd:verify-work` and `/gsd:complete-milestone` as alternatives. If `human_needed`, present the checklist and re-route based on approval. Recommend `/clear` first for fresh context in all cases.
