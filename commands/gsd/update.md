---
name: gsd:update
description: Update GSD to latest version with changelog display
---

# Update

Checks for GSD updates, installs if available, and displays what changed.

Read the installed version from ~/.claude/get-shit-done/VERSION. If the file is missing, treat the version as 0.0.0 and proceed to install. Check the latest published version via npm view get-shit-done-cc version. If npm is unavailable, tell the user to update manually with npx get-shit-done-cc --global and stop.

Compare versions. If installed equals latest, report up to date and stop. If installed is ahead of latest, note this (likely a development version) and stop.

If an update is available, fetch the changelog and extract entries between the installed and latest versions. Display the version diff and changelog entries, then warn that the installer performs a clean install — ~/.claude/commands/gsd/ and ~/.claude/get-shit-done/ will be wiped and replaced, and ~/.claude/agents/gsd-* files will be replaced. Note that custom commands in other locations, non-gsd-prefixed agents, custom hooks, and CLAUDE.md files are preserved. Advise backing up any directly modified GSD files. Ask the user to confirm before proceeding; if they cancel, stop.

Run npx get-shit-done-cc --global to perform the update. If it fails, show the error and stop. On success, clear the update cache by removing ~/.claude/cache/gsd-update-check.json. Display the version transition and remind the user to restart Claude Code to pick up the new commands.
