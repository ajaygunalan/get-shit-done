---
name: set-profile
description: Switch model profile for GSD agents (quality/balanced/budget)
arguments:
  - name: profile
    description: "Profile name: quality, balanced, or budget"
    required: true
---

# Set Profile

Switches the model profile used by GSD agents, controlling the quality-vs-cost tradeoff.

| Profile | Description |
|---------|-------------|
| quality | Opus everywhere except read-only verification |
| balanced | Opus for planning, Sonnet for execution/verification (default) |
| budget | Sonnet for writing, Haiku for research/verification |

Validate that the argument is one of `quality`, `balanced`, or `budget`; if not, error with the valid options and stop. Read `.planning/config.json`; if it doesn't exist, error and tell the user to run `/gsd:new-project`. Update the `model_profile` field in config.json (add it if missing) and write the file back. Confirm the new profile and display the agent-to-model mapping table for the selected profile. Note that the next spawned agents will use the new profile.
