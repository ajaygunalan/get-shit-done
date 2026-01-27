---
name: gsd:audit-milestone
description: Audit milestone completion against original intent before archiving
argument-hint: "[version]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Task
  - Write
---

# Audit Milestone

Verifies a milestone achieved its definition of done by checking requirements coverage, cross-phase integration, and end-to-end flows. This command is the orchestrator: it reads existing VERIFICATION.md files (phases already verified during execute-phase), aggregates tech debt and deferred gaps, then spawns an integration checker for cross-phase wiring.

@.planning/PROJECT.md
@.planning/REQUIREMENTS.md
@.planning/ROADMAP.md
@.planning/config.json (if exists)

Read model profile from .planning/config.json, defaulting to "balanced" if unset.

| Agent | quality | balanced | budget |
|-------|---------|----------|--------|
| gsd-integration-checker | sonnet | sonnet | haiku |

Parse the version from $ARGUMENTS or detect the current milestone from ROADMAP.md. Identify all phase directories in scope, extract the milestone definition of done from ROADMAP.md, and extract requirements mapped to this milestone from REQUIREMENTS.md.

Read each phase's VERIFICATION.md (glob `.planning/phases/*/*-VERIFICATION.md`). From each, extract: status (passed or gaps_found), critical gaps (blockers), non-critical gaps (tech debt, deferred items, warnings), anti-patterns (TODOs, stubs, placeholders), and requirements coverage. Flag any phase missing a VERIFICATION.md as "unverified phase" -- this is a blocker.

Spawn a gsd-integration-checker agent with the collected phase context (phase directories, exports from summaries, API routes created). It verifies cross-phase wiring and E2E user flows. Combine its report with the phase-level gaps and tech debt from the verification files.

Check each requirement in REQUIREMENTS.md mapped to this milestone: find its owning phase, check that phase's verification status, and determine whether the requirement is satisfied, partial, or unsatisfied.

Write `.planning/v{version}-MILESTONE-AUDIT.md` with YAML frontmatter containing: milestone version, audit timestamp, overall status, scores (requirements, phases, integration, flows as N/M), structured gaps (requirements, integration, flows), and tech_debt grouped by phase. Follow with a full markdown report with tables. Status is `passed` if all requirements are met with no critical gaps, `gaps_found` if critical blockers exist, or `tech_debt` if no blockers but accumulated deferred items need review.

Route the final output by status. If passed: show the score and report path, direct to `/gsd:complete-milestone {version}`. If gaps_found: show unsatisfied requirements (by REQ-ID, phase, and reason), cross-phase issues, and broken flows, then direct to `/gsd:plan-milestone-gaps`. If tech_debt: show all requirements met with accumulated debt listed by phase and item count, then offer two options -- complete the milestone (accept debt, track in backlog) via /gsd:complete-milestone, or plan a cleanup phase via /gsd:plan-milestone-gaps. Suggest `/clear` before the next command for a fresh context window.
