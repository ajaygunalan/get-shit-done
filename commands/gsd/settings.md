---
name: gsd:settings
description: Configure GSD workflow toggles and model profile
allowed-tools:
  - Read
  - Write
  - AskUserQuestion
---

# Settings

Interactive configuration for GSD workflow agents and model profile. Updates `.planning/config.json`.

Read `.planning/config.json` first; if it doesn't exist, error and tell the user to run `/gsd:new-project`. Parse current values for `workflow.research`, `workflow.plan_check`, `workflow.verifier` (default `true` if absent), and `model_profile` (default `balanced`).

Present four settings via AskUserQuestion, pre-selecting current values. First, model profile: quality (Opus everywhere except verification, highest cost), balanced (Opus for planning, Sonnet for execution/verification), or budget (Sonnet for writing, Haiku for research/verification). Then three yes/no toggles: spawn Plan Researcher (researches domain before planning), spawn Plan Checker (verifies plans before execution), and spawn Execution Verifier (verifies phase completion).

Merge selections into `config.json`, writing `model_profile` and the `workflow` object with `research`, `plan_check`, and `verifier` booleans. Confirm with a table showing all four settings and their new values. Mention that these apply to future `/gsd:plan-phase` and `/gsd:execute-phase` runs, and note the quick commands `/gsd:set-profile <profile>`, `/gsd:plan-phase --research`, `/gsd:plan-phase --skip-research`, and `/gsd:plan-phase --skip-verify`.
