---
name: gsd:help
description: Show available GSD commands and usage guide
---

Display the complete GSD command reference below. Output ONLY this reference content -- no project-specific analysis, git status, next-step suggestions, or commentary.

# GSD Command Reference

GSD (Get Shit Done) creates hierarchical project plans optimized for solo agentic development with Claude Code. Update periodically with `npx get-shit-done-cc@latest`.

Core loop: `/gsd:new-project` then `/gsd:plan-phase` then `/gsd:execute-phase`, repeat.

## Project Initialization

`/gsd:new-project` -- Initialize new project through unified flow: deep questioning, optional domain research (4 parallel researcher agents), requirements with v1/v2/out-of-scope scoping, roadmap with phase breakdown and success criteria. Creates .planning/ artifacts: PROJECT.md, config.json, research/, REQUIREMENTS.md (with REQ-IDs), ROADMAP.md, STATE.md.

`/gsd:map-codebase` -- Analyze existing codebase with parallel Explore agents. Creates .planning/codebase/ with 7 documents covering stack, architecture, structure, conventions, testing, integrations, concerns. Use before /gsd:new-project on existing codebases.

## Phase Planning

`/gsd:discuss-phase <number>` -- Capture how you imagine a phase working. Creates CONTEXT.md with vision, essentials, and boundaries. Use when you have ideas about how something should look or feel.

`/gsd:research-phase <number>` -- Ecosystem research for niche/complex domains. Creates RESEARCH.md with "how experts build this" knowledge -- standard stack, architecture patterns, pitfalls. For 3D, games, audio, shaders, ML, and other specialized domains.

`/gsd:list-phase-assumptions <number>` -- See Claude's intended approach for a phase before planning starts. Conversational output only, no files created.

`/gsd:plan-phase <number>` -- Create detailed execution plan. Generates .planning/phases/XX-phase-name/XX-YY-PLAN.md with concrete tasks, verification criteria, and success measures. Multiple plans per phase supported. Flags: `--research`, `--skip-research`, `--gaps`, `--skip-verify`.

## Execution

`/gsd:execute-phase <phase-number>` -- Execute all plans in a phase. Groups plans by wave (from frontmatter), executes waves sequentially, plans within each wave run in parallel via Task tool. Verifies phase goal after all plans complete. Updates REQUIREMENTS.md, ROADMAP.md, STATE.md.

## Task Cycle

`/gsd:task [description]` -- Full quality pipeline for a single ad-hoc task: discuss, plan, execute, verify. Creates artifacts in .planning/tasks/NNN-slug/. Uses quality agents (researcher, plan checker, verifier). Supports iteration. Does not modify ROADMAP.md.

## Quick Mode

`/gsd:quick` -- Small ad-hoc tasks with GSD guarantees but shorter path: spawns planner + executor, skips researcher/checker/verifier. Lives in .planning/quick/NNN-slug/. Updates STATE.md but not ROADMAP.md.

## Roadmap Management

`/gsd:add-phase <description>` -- Append new phase to end of current milestone in ROADMAP.md with next sequential number.

`/gsd:insert-phase <after> <description>` -- Insert urgent work as decimal phase between existing phases (e.g., 7.1 between 7 and 8).

`/gsd:remove-phase <number>` -- Remove a future (unstarted) phase and renumber subsequent phases to close the gap. Deletes phase directory and all references.

## Milestone Management

`/gsd:new-milestone <name>` -- Start new milestone through unified flow (mirrors /gsd:new-project for brownfield projects with existing PROJECT.md): questioning, optional research, requirements, roadmap.

`/gsd:complete-milestone <version>` -- Archive completed milestone: creates MILESTONES.md entry with stats, archives to milestones/ directory, creates git tag, prepares workspace for next version.

## Progress & Sessions

`/gsd:progress` -- Visual progress bar, completion percentage, recent work summary, current position, key decisions, open issues. Routes to next action (execute next plan or create missing plan). Detects 100% milestone completion.

`/gsd:resume-work` -- Restore context from previous session via STATE.md. Shows current position, recent progress, and next actions.

`/gsd:pause-work` -- Create .continue-here file with current state, update STATE.md session continuity section, capture in-progress work context.

## Debugging

`/gsd:debug [issue description]` -- Systematic debugging with persistent state in .planning/debug/[slug].md. Gathers symptoms through adaptive questioning, investigates using scientific method (evidence, hypothesis, test). Survives /clear -- run /gsd:debug with no args to resume. Archives resolved issues to .planning/debug/resolved/.

## Todo Management

`/gsd:add-todo [description]` -- Capture idea or task as structured todo in .planning/todos/pending/. Extracts context from conversation if no description given. Infers area from file paths, checks for duplicates, updates STATE.md todo count.

`/gsd:check-todos [area]` -- List pending todos with title, area, age. Optional area filter. Loads full context for selected todo, routes to appropriate action (work now, add to phase, brainstorm), moves todo to done/ when work begins.

## Verification & Auditing

`/gsd:verify-work [phase]` -- Conversational UAT: extracts testable deliverables from SUMMARY.md files, presents tests one at a time, diagnoses failures, creates fix plans ready for re-execution.

`/gsd:audit-milestone [version]` -- Reads all phase VERIFICATION.md files, checks requirements coverage, spawns integration checker for cross-phase wiring, creates MILESTONE-AUDIT.md with gaps and tech debt.

`/gsd:plan-milestone-gaps` -- Reads MILESTONE-AUDIT.md, groups gaps into phases prioritized by requirement priority (must/should/nice), adds gap closure phases to ROADMAP.md ready for /gsd:plan-phase.

## Configuration

`/gsd:settings` -- Interactive configuration of workflow toggles (researcher, plan checker, verifier agents) and model profile. Updates .planning/config.json.

`/gsd:set-profile <profile>` -- Quick model profile switch. `quality`: Opus everywhere except verification. `balanced` (default): Opus for planning, Sonnet for execution. `budget`: Sonnet for writing, Haiku for research/verification.

## Utility

`/gsd:help` -- This reference.

`/gsd:update` -- Version comparison (installed vs latest), changelog for missed versions, breaking change highlights, confirms before installing.

`/gsd:join-discord` -- Join the GSD Discord community for help and updates.

## Planning Configuration

.planning/config.json controls workflow behavior. `planning.commit_docs` (default true): when false, add .planning/ to .gitignore to keep planning artifacts local-only -- useful for OSS contributions, client projects, or private planning. `planning.search_gitignored` (default false): when true, adds --no-ignore to broad ripgrep searches so gitignored .planning/ files are still searchable.

## Workflow Modes

Set during /gsd:new-project. Interactive mode confirms each major decision and pauses at checkpoints. YOLO mode auto-approves most decisions and only stops for critical checkpoints. Change anytime by editing .planning/config.json.

## Quick Reference

Read .planning/PROJECT.md for project vision, .planning/STATE.md for current context, .planning/ROADMAP.md for phase status. Run /gsd:progress to check where you are.
