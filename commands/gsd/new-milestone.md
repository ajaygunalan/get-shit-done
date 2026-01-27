---
name: gsd:new-milestone
description: Start a new milestone cycle — update PROJECT.md and route to requirements
argument-hint: "[milestone name, e.g., 'v1.1 Notifications']"
allowed-tools:
  - Read
  - Write
  - Bash
  - Task
  - AskUserQuestion
---

# New Milestone

Starts a new milestone through a unified flow: questioning, research (optional), requirements, roadmap. This is the brownfield equivalent of /gsd:new-project -- the project exists, PROJECT.md has history. Creates or updates `.planning/PROJECT.md`, `.planning/research/`, `.planning/REQUIREMENTS.md`, `.planning/ROADMAP.md`, and `.planning/STATE.md`. After this command, run `/gsd:plan-phase [N]`.

@~/.claude/get-shit-done/references/questioning.md
@~/.claude/get-shit-done/references/ui-brand.md
@~/.claude/get-shit-done/templates/project.md
@~/.claude/get-shit-done/templates/requirements.md

Milestone name comes from $ARGUMENTS (prompt if not provided).

## Load Context

Read `.planning/PROJECT.md` (existing project, Validated requirements, decisions), `.planning/MILESTONES.md` (what shipped previously), `.planning/STATE.md` (pending todos, blockers), and `.planning/config.json`. Check for `.planning/MILESTONE-CONTEXT.md` (from /gsd:discuss-milestone).

## Gather Milestone Goals

If MILESTONE-CONTEXT.md exists, use the features and scope from discuss-milestone and present a summary for confirmation. If it does not exist, present what shipped in the last milestone, ask "What do you want to build next?", and use AskUserQuestion to explore features, priorities, constraints, and scope.

## Determine Milestone Version

Parse the last version from MILESTONES.md, suggest the next version (v1.0 to v1.1, or v2.0 for major changes), and confirm with the user.

## Update PROJECT.md and STATE.md

Add a "Current Milestone: v[X.Y] [Name]" section to PROJECT.md with the milestone goal and target features. Update Active requirements with new goals and the "Last updated" footer. Reset STATE.md to "Not started (defining requirements)" status with today's date, but keep the Accumulated Context section (decisions, blockers) from the previous milestone.

## Cleanup and Commit

Delete MILESTONE-CONTEXT.md if it exists (consumed). Check commit_docs in `.planning/config.json` (default true; also false if .planning is gitignored). If committing, add PROJECT.md and STATE.md and commit.

## Resolve Model Profile

Read model_profile from `.planning/config.json` (default "balanced") and resolve agent models using this table:

| Agent | quality | balanced | budget |
|-------|---------|----------|--------|
| gsd-project-researcher | opus | sonnet | haiku |
| gsd-research-synthesizer | sonnet | sonnet | haiku |
| gsd-roadmapper | opus | sonnet | sonnet |

## Research

Ask via AskUserQuestion whether to research the domain ecosystem for new features before defining requirements or skip. If researching, create `.planning/research/` and spawn 4 parallel gsd-project-researcher agents (using resolved researcher model), each milestone-aware -- they receive the list of Validated requirements from PROJECT.md so they focus only on what is needed for the new features, not re-researching existing capabilities.

The four agents write to `.planning/research/`: STACK.md (stack additions and integration points for new features), FEATURES.md (table stakes vs differentiators vs anti-features for the new capabilities, with dependencies on existing features), ARCHITECTURE.md (how new features integrate with existing architecture, new vs modified components, build order), and PITFALLS.md (common mistakes when adding these features to an existing system, prevention strategies). Each agent reads `~/.claude/agents/gsd-project-researcher.md` for instructions.

After all 4 complete, spawn a gsd-research-synthesizer agent (using resolved synthesizer model) to read all four research files and write `.planning/research/SUMMARY.md`, then commit. Present key findings.

## Define Requirements

Load PROJECT.md to extract core value, current milestone goals, and validated requirements. If research exists, load research/FEATURES.md and present features by category. If no research, gather requirements conversationally by asking what users need to do with the new features, then clarifying and grouping.

For each category, use AskUserQuestion with multiSelect to let the user choose features for this milestone. Selected features become this milestone's requirements; unselected table stakes go to a future milestone; unselected differentiators go to out-of-scope. Ask whether research missed anything.

Generate `.planning/REQUIREMENTS.md` with this milestone's requirements grouped by category (checkboxes, REQ-IDs in `[CATEGORY]-[NUMBER]` format like AUTH-01, NOTIF-02), continuing numbering from existing requirements if applicable. Include future requirements, out-of-scope exclusions with reasoning, and an empty traceability section. Requirements must be specific/testable, user-centric, atomic, and independent. Present the full list for confirmation; loop back if the user wants adjustments. Check commit_docs config and commit if enabled.

## Create Roadmap

Determine the starting phase number by reading MILESTONES.md to find the last phase from the previous milestone (e.g., if v1.0 ended at phase 5, v1.1 starts at phase 6). Spawn a gsd-roadmapper agent (using resolved roadmapper model) with PROJECT.md, REQUIREMENTS.md, research/SUMMARY.md (if exists), config.json, and MILESTONES.md as context. The roadmapper derives phases from this milestone's requirements only (not validated/existing), continues phase numbering, maps every requirement to one phase, derives 2-5 success criteria per phase, validates 100% coverage, and writes ROADMAP.md, STATE.md, and updates REQUIREMENTS.md traceability immediately.

If the roadmapper returns ROADMAP BLOCKED, present the blocker and work with the user to resolve, then re-spawn. If ROADMAP CREATED, read ROADMAP.md and present it inline with phases, goals, requirement mappings, and success criteria.

Ask via AskUserQuestion whether to approve, adjust phases, or review the full file. If adjusting, capture notes and re-spawn the roadmapper with revision context, looping until approved. If reviewing, display the raw file and re-ask. After approval, check commit_docs config and commit ROADMAP.md, STATE.md, and REQUIREMENTS.md if enabled.

## Done

Present completion showing milestone name and version, all artifact locations, phase and requirement counts, and next steps: `/gsd:discuss-phase [N]` to gather context, or `/gsd:plan-phase [N]` to plan directly. Recommend `/clear` first for a fresh context window.
