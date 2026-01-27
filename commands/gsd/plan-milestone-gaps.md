---
name: gsd:plan-milestone-gaps
description: Create phases to close all gaps identified by milestone audit
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# Plan Milestone Gaps

Creates all phases necessary to close gaps identified by /gsd:audit-milestone. One command creates all fix phases -- no manual /gsd:add-phase per gap.

@.planning/PROJECT.md
@.planning/REQUIREMENTS.md
@.planning/ROADMAP.md
@.planning/STATE.md

Glob for `.planning/v*-MILESTONE-AUDIT.md` and use the most recent. Parse its YAML frontmatter to extract structured gaps: `gaps.requirements` (unsatisfied requirements), `gaps.integration` (missing cross-phase connections), and `gaps.flows` (broken E2E flows). If no audit file exists or it contains no gaps, error and direct the user to run /gsd:audit-milestone first.

Prioritize gaps using REQUIREMENTS.md priority levels: `must` gaps get a phase and block the milestone, `should` gaps get a phase and are recommended, `nice` gaps are presented to the user to include or defer. For integration and flow gaps, infer priority from the affected requirements.

Group related gaps into logical phases. Combine gaps affecting the same phase, same subsystem (auth, API, UI), or sharing dependency order (fix stubs before wiring). Keep phases focused at 2-4 tasks each. Determine phase numbers by continuing from the highest existing phase directory.

Present the gap closure plan to the user showing: milestone version, total gap counts by type, each proposed phase with its name, the gaps it closes (by REQ-ID and description), and task count. If nice-to-have gaps exist, list them separately and ask whether to include or defer. Wait for user confirmation before proceeding.

Update ROADMAP.md with new phase entries, each including a goal derived from the gaps being closed, the REQ-IDs being satisfied, and a "Gap Closure" annotation. Create corresponding phase directories under `.planning/phases/`. Check .planning/config.json for `commit_docs` setting (default true) and whether `.planning` is gitignored; if committing is enabled, stage ROADMAP.md and commit with message `docs(roadmap): add gap closure phases {N}-{M}`.

After completion, direct the user to run `/gsd:plan-phase {N}` for the first gap closure phase (suggest `/clear` first for a fresh context window), and note that after all gap phases complete they should re-audit with /gsd:audit-milestone then archive with /gsd:complete-milestone.
