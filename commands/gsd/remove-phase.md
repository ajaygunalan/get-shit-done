---
name: gsd:remove-phase
description: Remove a future phase from roadmap and renumber subsequent phases
argument-hint: <phase-number>
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

# Remove Phase

Remove an unstarted future phase from the roadmap and renumber all subsequent phases to maintain a clean, linear sequence. The git commit serves as the historical record -- no "removed phase" notes in STATE.md.

@.planning/ROADMAP.md
@.planning/STATE.md

Parse the phase number argument (integer or decimal, e.g., 17 or 16.1). If no argument provided, error with usage: `/gsd:remove-phase <phase-number>` and exit.

Read `.planning/STATE.md` and `.planning/ROADMAP.md`. Parse the current phase number from STATE.md. Verify the target phase exists in ROADMAP.md by searching for the `### Phase {target}:` heading -- if not found, list available phases and exit. Verify the target is a future phase (target > current phase number). If target <= current phase, error explaining only future phases can be removed and suggest /gsd:pause-work instead. Check for SUMMARY.md files in the phase directory -- if any exist, error explaining phases with completed work cannot be removed, list the SUMMARY.md files found, and exit.

Collect the phase name from the ROADMAP.md heading, the phase directory path `.planning/phases/{target}-{slug}/`, and identify all subsequent phases needing renumbering. For integer phase removal (e.g., 17): find all phases > 17, all decimal phases >= 17.0 and < 18.0 (these become 16.x), and all decimal phases under subsequent integers. For decimal phase removal (e.g., 17.1): find only other decimals in the same series (17.2 becomes 17.1, 17.3 becomes 17.2) -- integer phases stay unchanged.

Present a removal summary showing what will be deleted and the full renumbering plan, then wait for user confirmation.

Delete the target phase directory if it exists. Rename all subsequent phase directories in descending order to avoid conflicts (20 to 19, then 19 to 18, then 18 to 17). Rename decimal directories accordingly. Inside each renumbered directory, rename files containing the old phase number (e.g., `18-01-PLAN.md` becomes `17-01-PLAN.md`). CONTEXT.md and DISCOVERY.md have no phase prefix, so they need no rename.

Update ROADMAP.md: remove the target phase section entirely (heading through next heading), remove from the phase list, remove from the progress table, renumber all subsequent phase headings and list entries and table rows and plan references, update dependency references (phases that depended on the removed phase should point to the prior phase), and renumber decimal phases under the removed integer. Write the updated ROADMAP.md.

Update STATE.md: update the total phase count and recalculate the progress percentage. Write updated STATE.md. Search for and update phase references inside plan files in renumbered directories.

Check `.planning/config.json` for `commit_docs` setting (default: true). Also check if `.planning` is gitignored. If committing is enabled, stage `.planning/` and commit with message `chore: remove phase {target} ({original-phase-name})`. If disabled or gitignored, skip git operations.

Present the completion summary showing what was deleted, what was renumbered, the updated file list, the commit message, the remaining phase count, and current position. Offer /gsd:progress to see updated status.
