---
name: gsd:plan-phase
description: Create detailed execution plan for a phase (PLAN.md) with verification loop
argument-hint: "[phase] [--research] [--skip-research] [--gaps] [--skip-verify]"
agent: gsd-planner
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - WebFetch
  - mcp__context7__*
---

@~/.claude/get-shit-done/references/ui-brand.md

# Plan Phase

Creates executable phase prompts (PLAN.md files) for a roadmap phase. Default flow: Research (if needed) then Plan then Verify. The orchestrator parses arguments, validates the phase, handles research, spawns the gsd-planner agent, verifies plans with gsd-plan-checker, and iterates until plans pass or max iterations are reached. Subagents are used because research and planning burn context fast; verification needs a fresh window.

$ARGUMENTS provides the phase number (optional -- auto-detects next unplanned phase if omitted). Flags: `--research` forces re-research even if RESEARCH.md exists, `--skip-research` skips research entirely, `--gaps` enters gap closure mode (reads VERIFICATION.md, skips research), `--skip-verify` bypasses the planner-checker verification loop.

Validate that .planning/ exists; if not, error telling the user to run /gsd:new-project first. Resolve the model profile from .planning/config.json (default "balanced"):

| Agent | quality | balanced | budget |
|-------|---------|----------|--------|
| gsd-phase-researcher | opus | sonnet | haiku |
| gsd-planner | opus | opus | sonnet |
| gsd-plan-checker | sonnet | sonnet | haiku |

Parse the phase number (integer or decimal like 2.1) and flags from $ARGUMENTS. If no phase number is given, detect the next unplanned phase from the roadmap. Normalize to zero-padded format (8 becomes 08, 2.1 becomes 02.1). Check for existing research and plans under .planning/phases/${PHASE}-*/.

Validate the phase against .planning/ROADMAP.md. If not found, error with available phases. Extract phase number, name, and description. Ensure the phase directory exists under .planning/phases/; create it from the roadmap name if missing.

For research: skip if `--gaps` is set (gap closure uses VERIFICATION.md instead). Skip if `--skip-research` is set. Check .planning/config.json for `workflow.research`; if false and `--research` is not set, skip. If RESEARCH.md already exists and `--research` is not set, use existing research and skip. Otherwise spawn gsd-phase-researcher. Gather context from the roadmap phase description, REQUIREMENTS.md, STATE.md decisions, and any CONTEXT.md in the phase directory. Send the researcher a prompt asking "What do I need to know to PLAN this phase well?" with all gathered context, and direct output to {phase_dir}/{phase}-RESEARCH.md. Spawn via Task with the researcher model, instructing it to first read ~/.claude/agents/gsd-phase-researcher.md. On return: if RESEARCH COMPLETE, proceed to planning. If RESEARCH BLOCKED, display blocker information and offer the user three choices: provide more context, skip research and plan anyway, or abort.

Check for existing plans in the phase directory. If plans exist, offer the user three choices: continue planning (add more plans), view existing plans, or replan from scratch.

Read all context files for the planner agent -- the @ syntax does not work across Task() boundaries so content must be inlined. Required: STATE.md, ROADMAP.md. Optional: REQUIREMENTS.md, CONTEXT.md, RESEARCH.md. In --gaps mode also read VERIFICATION.md and UAT.md.

Spawn gsd-planner via Task with the planner model, instructing it to first read ~/.claude/agents/gsd-planner.md. Provide inlined context including phase number, mode (standard or gap_closure), all file contents gathered above, and note that output is consumed by /gsd:execute-phase -- plans must be executable prompts with frontmatter (wave, depends_on, files_modified, autonomous), XML-format tasks, verification criteria, and must_haves for goal-backward verification.

On planner return: if PLANNING COMPLETE, display plan count. If `--skip-verify` is set or .planning/config.json `workflow.plan_check` is false, skip verification and go to final status. Otherwise proceed to verification. If CHECKPOINT REACHED, present to user, get response, and spawn continuation. If PLANNING INCONCLUSIVE, show what was attempted and offer: add context, retry, or manual planning.

For verification, spawn gsd-plan-checker via Task with the checker model using the gsd-plan-checker subagent type. Provide all plans from the phase directory and REQUIREMENTS.md. The checker returns either VERIFICATION PASSED or ISSUES FOUND. If passed, proceed to final status. If issues found, enter the revision loop.

The revision loop runs up to 3 iterations total. On each iteration, re-spawn gsd-planner in revision mode with existing plans and the checker's structured issues, instructing targeted updates rather than replanning from scratch. After the planner returns, re-spawn the checker. If verification passes, proceed to final status. If max iterations are reached with issues remaining, display the remaining issues and offer three choices: force proceed despite issues, provide guidance and retry, or abandon planning.

Final status displays a summary showing phase name, plan count, wave breakdown (table of wave number to plans to objectives), research status (completed, used existing, or skipped), and verification status (passed, passed with override, or skipped). Offer next steps: /gsd:execute-phase {X} (suggest /clear first for fresh context), and note that plans can be reviewed with cat .planning/phases/{phase-dir}/*-PLAN.md or re-researched with /gsd:plan-phase {X} --research.
