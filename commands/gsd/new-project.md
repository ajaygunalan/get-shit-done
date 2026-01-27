---
name: gsd:new-project
description: Initialize a new project with deep context gathering and PROJECT.md
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---

# New Project

Initializes a project through a unified flow: questioning, research (optional), requirements, roadmap. Creates `.planning/PROJECT.md`, `.planning/config.json`, `.planning/research/` (optional), `.planning/REQUIREMENTS.md`, `.planning/ROADMAP.md`, and `.planning/STATE.md`. After this command, run `/gsd:plan-phase 1`.

@~/.claude/get-shit-done/references/questioning.md
@~/.claude/get-shit-done/references/ui-brand.md
@~/.claude/get-shit-done/templates/project.md
@~/.claude/get-shit-done/templates/requirements.md

## Setup

Before any user interaction, run all of these checks. Abort with an error if `.planning/PROJECT.md` already exists (direct user to /gsd:progress). Initialize a git repo in the current directory if one does not exist. Detect brownfield projects by scanning for code files (ts, js, py, go, rs, swift, java), manifest files (package.json, requirements.txt, Cargo.toml, go.mod, Package.swift), and whether `.planning/codebase/` already exists.

If existing code is detected and `.planning/codebase/` does not exist, ask via AskUserQuestion whether to map the codebase first or skip. If the user chooses mapping, run /gsd:map-codebase and exit so they can return to /gsd:new-project afterward.

## Questioning

Open with a freeform inline question: "What do you want to build?" Do not use AskUserQuestion for this opener. After the user responds, follow threads with AskUserQuestion-based probes that dig into their response -- interpretations, clarifications, concrete examples. Consult questioning.md for techniques: challenge vagueness, make abstract concrete, surface assumptions, find edges, reveal motivation. Mentally track the context checklist from questioning.md and weave missing items naturally into the conversation rather than switching to checklist mode.

When you have enough to write a clear PROJECT.md, ask via AskUserQuestion whether to create it or keep exploring. Loop until the user chooses to proceed.

## Write PROJECT.md

Synthesize all context into `.planning/PROJECT.md` using templates/project.md. For greenfield projects, initialize all requirements as Active hypotheses with none Validated yet. For brownfield projects (codebase map exists), read `.planning/codebase/ARCHITECTURE.md` and `STACK.md` to infer Validated requirements from existing capabilities, then add new goals as Active. Include a Key Decisions table initialized from choices made during questioning. Add a "Last updated" footer with the date. Commit PROJECT.md immediately.

## Workflow Preferences

Gather preferences in two AskUserQuestion rounds. Round 1 covers four core settings: mode (YOLO auto-approve vs Interactive confirm-each-step), depth (Quick 3-5 phases / Standard 5-8 phases / Comprehensive 8-12 phases), parallelization (parallel vs sequential plan execution), and git tracking (commit planning docs vs keep .planning/ local-only via .gitignore).

Round 2 covers three workflow agents and model profile. The agents -- Researcher (investigates domain before planning), Plan Checker (verifies plan achieves phase goal), and Verifier (confirms deliverables after execution) -- each add tokens and time but improve quality; all recommended for important projects. Model profile selects which AI models power planning agents: Quality (Opus-heavy), Balanced (Sonnet-heavy, recommended), or Budget (Haiku where possible).

Write `.planning/config.json` with keys: mode, depth, parallelization, commit_docs, model_profile, and workflow (research, plan_check, verifier booleans). If commit_docs is false, add `.planning/` to .gitignore. Commit config.json. Settings can be changed later via /gsd:settings.

## Resolve Model Profile

Read model_profile from `.planning/config.json` (default "balanced") and resolve agent models using this table:

| Agent | quality | balanced | budget |
|-------|---------|----------|--------|
| gsd-project-researcher | opus | sonnet | haiku |
| gsd-research-synthesizer | sonnet | sonnet | haiku |
| gsd-roadmapper | opus | sonnet | sonnet |

## Research

Ask via AskUserQuestion whether to research the domain ecosystem before defining requirements or skip. If researching, create `.planning/research/` and determine milestone context: greenfield (no Validated requirements in PROJECT.md) or subsequent (Validated requirements exist).

Spawn 4 parallel gsd-project-researcher agents (using resolved researcher model), each writing to `.planning/research/`: STACK.md (standard stack for the domain, specific libraries with versions, what not to use), FEATURES.md (table stakes vs differentiators vs anti-features, complexity and dependencies), ARCHITECTURE.md (component boundaries, data flow, build order), and PITFALLS.md (domain-specific mistakes, warning signs, prevention strategies, phase mapping). Each agent reads `~/.claude/agents/gsd-project-researcher.md` for instructions and receives PROJECT.md context plus milestone context (greenfield vs subsequent).

After all 4 complete, spawn a gsd-research-synthesizer agent (using resolved synthesizer model) to read all four research files and write `.planning/research/SUMMARY.md`, then commit. Present key findings from the summary.

## Define Requirements

Load PROJECT.md to extract core value, constraints, and scope boundaries. If research exists, load research/FEATURES.md and present features organized by category. If no research, gather requirements conversationally by asking what users need to do, then clarifying and grouping responses.

For each category, use AskUserQuestion with multiSelect to let the user choose which features are in v1. Selected features become v1 requirements; unselected table stakes go to v2; unselected differentiators go to out-of-scope. Ask whether the user wants to add anything research missed. Cross-check requirements against the Core Value from PROJECT.md and surface gaps.

Generate `.planning/REQUIREMENTS.md` with v1 requirements grouped by category (checkboxes, REQ-IDs in `[CATEGORY]-[NUMBER]` format like AUTH-01, CONTENT-02), v2 deferred requirements, out-of-scope exclusions with reasoning, and an empty traceability section. Requirements must be specific and testable, user-centric ("User can X"), atomic (one capability each), and independent. Present the full list for user confirmation; loop back to scoping if the user wants adjustments. Commit REQUIREMENTS.md.

## Create Roadmap

Spawn a gsd-roadmapper agent (using resolved roadmapper model) with PROJECT.md, REQUIREMENTS.md, research/SUMMARY.md (if exists), and config.json as context. The roadmapper derives phases from requirements, maps every v1 requirement to exactly one phase, derives 2-5 success criteria per phase, validates 100% coverage, and writes ROADMAP.md, STATE.md, and updates REQUIREMENTS.md traceability immediately (not as draft).

If the roadmapper returns ROADMAP BLOCKED, present the blocker and work with the user to resolve it, then re-spawn. If it returns ROADMAP CREATED, read ROADMAP.md and present it inline showing phases, goals, requirement mappings, and success criteria.

Ask via AskUserQuestion whether to approve, adjust phases, or review the full file. If adjusting, capture notes and re-spawn the roadmapper with revision context, looping until approved. If reviewing, display the raw file and re-ask. After approval, commit ROADMAP.md, STATE.md, and REQUIREMENTS.md.

## Done

Present completion showing the project name, all artifact locations, phase and requirement counts, and next steps: `/gsd:discuss-phase 1` to gather context, or `/gsd:plan-phase 1` to plan directly. Recommend `/clear` first for a fresh context window.
