---
type: prompt
name: gsd:complete-milestone
description: Archive completed milestone and prepare for next version
argument-hint: <version>
allowed-tools:
  - Read
  - Write
  - Bash
---

# Complete Milestone

Marks milestone {{version}} complete, archives artifacts to `.planning/milestones/`, and prepares for the next milestone.

@~/.claude/get-shit-done/workflows/complete-milestone.md
@~/.claude/get-shit-done/templates/milestone-archive.md

Load `.planning/ROADMAP.md`, `.planning/REQUIREMENTS.md`, `.planning/STATE.md`, and `.planning/PROJECT.md`.

Before starting, check for `.planning/v{{version}}-MILESTONE-AUDIT.md`. If missing, recommend `/gsd:audit-milestone` first. If audit status is `gaps_found`, recommend `/gsd:plan-milestone-gaps` first. If audit status is `passed`, proceed.

Verify readiness by confirming all phases in the milestone have SUMMARY.md files. Present milestone scope and stats, then wait for user confirmation. Gather stats: count phases, plans, tasks; calculate git range, file changes, LOC; extract timeline from git log. Present summary and confirm. Read all phase SUMMARY.md files in the milestone range and extract 4-6 key accomplishments for user approval.

Archive the milestone by creating `.planning/milestones/v{{version}}-ROADMAP.md` with full phase details using the milestone-archive.md template, then collapse the ROADMAP.md entry to a one-line summary with a link. Archive requirements to `.planning/milestones/v{{version}}-REQUIREMENTS.md`, marking all requirements complete with outcomes noted (validated, adjusted, dropped), then delete `.planning/REQUIREMENTS.md` so the next milestone starts fresh. Update PROJECT.md with a "Current State" section showing the shipped version and a "Next Milestone Goals" section; for v1.1+ archive previous content in a `<details>` block.

Commit all changed files with message `chore: archive v{{version}} milestone`, create tag `git tag -a v{{version}} -m "[milestone summary]"`, and ask about pushing the tag. Offer `/gsd:new-milestone` as the next step. Always create archive files before updating or deleting originals.
