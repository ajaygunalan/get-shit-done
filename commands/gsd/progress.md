---
name: gsd:progress
description: Check project progress, show context, and route to next action (execute or plan)
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
  - SlashCommand
---

# Progress

Check project progress, summarize recent work, and route to the next action.

Use Bash (not Glob) to verify `.planning/` exists, since Glob respects .gitignore and `.planning/` is often gitignored. If no `.planning/` directory exists, tell the user to run /gsd:new-project and exit. If STATE.md is missing, same suggestion. If ROADMAP.md is missing but PROJECT.md exists, go to Route F (between milestones). If both ROADMAP.md and PROJECT.md are missing, suggest /gsd:new-project and exit.

Read `.planning/STATE.md` (position, decisions, issues), `.planning/ROADMAP.md` (phase structure), `.planning/PROJECT.md` (vision, requirements), and `.planning/config.json` (model_profile, workflow toggles). Find the 2-3 most recent SUMMARY.md files and extract what was accomplished, key decisions, and issues. From STATE.md, parse the current phase, plan number, and status. Calculate total plans, completed plans, and remaining plans. Note blockers. For phases without PLAN.md files, check if `{phase}-CONTEXT.md` exists. Count pending todos via `ls .planning/todos/pending/*.md 2>/dev/null | wc -l`. Check for active debug sessions via `ls .planning/debug/*.md 2>/dev/null | grep -v resolved | wc -l`.

Present a status report showing: project name, visual progress bar (e.g., 8/10 plans complete), model profile, recent work (1-line summaries from recent SUMMARY.md files), current position (phase N of total, plan M of phase-total, CONTEXT.md presence), key decisions from STATE.md, blockers/concerns, pending todo count (with /gsd:check-todos mention), active debug session count if > 0 (with /gsd:debug mention), and what's next from ROADMAP.

To determine routing, count PLAN.md files, SUMMARY.md files, and UAT.md files in the current phase directory. Check for UAT.md files with status "diagnosed" (gaps needing fixes). Route based on these counts: if diagnosed UAT gaps exist, go to Route E. If summaries < plans, unexecuted plans exist -- go to Route A. If summaries = plans and plans > 0, the phase is complete -- go to Step 3. If plans = 0, the phase needs planning -- go to Route B.

Route A (unexecuted plan exists): Find the first PLAN.md without a matching SUMMARY.md. Read its objective. Show the plan name and objective, suggest `/gsd:execute-phase {phase}`, and recommend `/clear` first for a fresh context window.

Route B (phase needs planning): Check if `{phase}-CONTEXT.md` exists in the phase directory. If it does, show the phase name and goal, note context is gathered, and suggest `/gsd:plan-phase {phase-number}`. If CONTEXT.md does not exist, suggest `/gsd:discuss-phase {phase}` to gather context, with `/gsd:plan-phase {phase}` and `/gsd:list-phase-assumptions {phase}` as alternatives. Recommend `/clear` first in either case.

Route E (UAT gaps need fix plans): Show the count of gaps from the diagnosed UAT.md and suggest `/gsd:plan-phase {phase} --gaps`. Also mention `/gsd:execute-phase {phase}` and `/gsd:verify-work {phase}` as alternatives.

Step 3 (phase complete -- check milestone status): Read ROADMAP.md to find the current phase number and all phase numbers in the milestone. If the current phase < the highest phase, go to Route C. If current phase = highest phase, go to Route D.

Route C (phase complete, more phases remain): Show phase completion, then suggest `/gsd:discuss-phase {Z+1}` for the next phase. Offer `/gsd:plan-phase {Z+1}` to skip discussion and `/gsd:verify-work {Z}` for acceptance testing.

Route D (milestone complete): Show all phases finished, suggest `/gsd:complete-milestone`. Offer `/gsd:verify-work` for acceptance testing before completing.

Route F (between milestones -- ROADMAP.md missing, PROJECT.md exists): A milestone was completed and archived. Read MILESTONES.md for the last completed version, then suggest `/gsd:new-milestone` to start the next milestone cycle.

Handle edge cases inline: if a handoff file exists, mention it and offer `/gsd:resume-work`. If blockers are present, highlight them before offering to continue.
