---
name: gsd:pause-work
description: Create context handoff when pausing work mid-phase
allowed-tools:
  - Read
  - Write
  - Bash
---

# Pause Work

Creates a `.continue-here.md` handoff file so a fresh session can resume with full context.

@.planning/STATE.md

Detect the current phase directory from most recently modified files. Gather complete state for the handoff: current position (phase, plan, task), work completed this session, work remaining in the current plan/phase, decisions made with rationale, blockers or issues, mental context (approach, next steps), and files modified but not committed. Ask the user for clarifications if needed.

Write the handoff to `.planning/phases/XX-name/.continue-here.md` with YAML frontmatter containing phase, task number, total tasks, status, and timestamp, followed by sections for current state, completed work (per-task), remaining work, decisions made, blockers, context, and a specific next action for resuming. Be specific enough for a fresh Claude to understand immediately.

Check `.planning/config.json` for `commit_docs` and whether `.planning` is gitignored. If `commit_docs` is not `false` and `.planning` is not gitignored, stage `.planning/phases/*/.continue-here.md` and commit with message `wip: [phase-name] paused at task [X]/[Y]`. Otherwise skip git operations.

Confirm by showing the handoff file path, current phase, task progress, and status. Tell the user to run `/gsd:resume-work` to continue.
