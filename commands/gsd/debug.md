---
name: gsd:debug
description: Systematic debugging with persistent state across context resets
argument-hint: [issue description]
allowed-tools:
  - Read
  - Bash
  - Task
  - AskUserQuestion
---

# Debug

Debugs issues using scientific method with subagent isolation. The orchestrator gathers symptoms, spawns a gsd-debugger agent, handles checkpoints, and spawns continuations. Investigation burns context fast, so each round gets a fresh 200k window while the main context stays lean.

Read the model profile from .planning/config.json (default "balanced") and resolve the debugger model.

| Agent | quality | balanced | budget |
|-------|---------|----------|--------|
| gsd-debugger | opus | sonnet | sonnet |

Check for active debug sessions at .planning/debug/*.md (excluding resolved). If active sessions exist and no $ARGUMENTS was provided, list them with status, hypothesis, and next action so the user can pick one to resume or describe a new issue. If $ARGUMENTS is provided or the user describes a new issue, proceed to symptom gathering.

For a new issue, ask the user via AskUserQuestion for: expected behavior, actual behavior, error messages, timeline (when it started, whether it ever worked), and reproduction steps. Once all symptoms are gathered, confirm readiness to investigate.

Spawn the gsd-debugger agent via Task with the resolved model. Pass the issue slug, trigger summary, and all symptoms with mode set to find_and_fix and symptoms_prefilled true. The agent creates its debug file at .planning/debug/{slug}.md.

On agent return: if ROOT CAUSE FOUND, display the root cause and evidence summary, then offer "Fix now" (spawn a fix subagent), "Plan fix" (suggest /gsd:plan-phase --gaps), or "Manual fix" (done). If CHECKPOINT REACHED, present the checkpoint to the user, get their response, and spawn a fresh continuation agent with the prior debug file reference (@.planning/debug/{slug}.md) and the checkpoint response, using the same debugger model. If INVESTIGATION INCONCLUSIVE, show what was checked and eliminated, then offer to continue investigating with a new agent, investigate manually, or add more context and spawn again.
