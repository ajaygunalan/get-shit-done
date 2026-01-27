---
name: gsd:discuss-phase
description: Gather phase context through adaptive questioning before planning
argument-hint: "<phase>"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# Discuss Phase

Extracts implementation decisions that downstream agents (researcher, planner) need so they can act without re-asking the user. Output: `{phase}-CONTEXT.md` in .planning/.

@~/.claude/get-shit-done/workflows/discuss-phase.md
@~/.claude/get-shit-done/templates/context.md

Load @.planning/STATE.md and @.planning/ROADMAP.md. Validate the phase number from $ARGUMENTS — error if missing or absent from roadmap. If CONTEXT.md already exists for this phase, offer to update, view, or skip.

Analyze the phase goal to identify its domain — something users see (layout, density, interactions, states), call (responses, errors, auth, versioning), run (output format, flags, modes, error handling), read (structure, tone, depth, flow), or being organized (criteria, grouping, naming, exceptions). Generate 3–4 phase-specific gray areas from that domain, not generic categories. Present them as a multi-select — no skip option. For each selected area, ask 4 probing questions, then offer "more questions about [area], or move to next?" — if more, ask 4 more and check again. Do not ask about technical implementation, architecture, performance, or scope expansion — Claude handles those.

The phase boundary from ROADMAP.md is fixed. Discussion clarifies how to implement, not whether to add more. If the user suggests new capabilities, respond "That's its own phase. I'll note it for later." Capture deferred ideas without acting on them.

After all areas are explored, write CONTEXT.md with sections matching the areas discussed, capturing concrete decisions rather than vague vision. Offer next steps: research or plan.
