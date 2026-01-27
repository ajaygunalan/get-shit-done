---
name: gsd:research-phase
description: Research how to implement a phase (standalone - usually use /gsd:plan-phase instead)
argument-hint: "[phase]"
allowed-tools:
  - Read
  - Bash
  - Task
---

# Research Phase

Spawns a gsd-phase-researcher agent to investigate how to implement a phase. This is standalone — /gsd:plan-phase integrates research automatically. Use this when you want research without planning, want to re-research after planning, or need to investigate feasibility before committing to a phase.

Read the model profile from .planning/config.json (default "balanced") and resolve the researcher model.

| Agent | quality | balanced | budget |
|-------|---------|----------|--------|
| gsd-phase-researcher | opus | sonnet | haiku |

$ARGUMENTS is the phase number (required). Normalize it (8 to 08, 2.1 to 02.1) then validate against .planning/ROADMAP.md. If not found, error and exit. If found, extract the phase number, name, and description.

Check for existing research at .planning/phases/${PHASE}-*/RESEARCH.md. If it exists, ask the user whether to update it, view it, or skip. If it does not exist, continue.

Gather phase context: the ROADMAP.md entry (grep -A20), REQUIREMENTS.md, any phase-specific *-CONTEXT.md files, and decisions from STATE.md. Present a summary before spawning.

Spawn the gsd-phase-researcher agent via Task with the researcher model. Pass the phase description, requirements, prior decisions, and phase context. The research mode defaults to ecosystem (other modes: feasibility, implementation, comparison). The agent's core job is discovering unknowns — established architecture patterns, standard stack, common pitfalls, SOTA approaches, and what should not be hand-rolled. Direct it to be prescriptive ("Use X" not "Consider X or Y") because its RESEARCH.md feeds directly into /gsd:plan-phase which expects specific sections: Standard Stack, Architecture Patterns, Don't Hand-Roll, Common Pitfalls, Code Examples. Output goes to .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md.

On agent return: if RESEARCH COMPLETE, display a summary and offer to plan the phase, dig deeper, review the full output, or finish. If CHECKPOINT REACHED, present the checkpoint to the user, get their response, and spawn a continuation agent with the prior research file and checkpoint response. If RESEARCH INCONCLUSIVE, show what was attempted and offer to add context, try a different mode, or investigate manually.
