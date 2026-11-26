---
name: draft-release-notes
description: "Draft release notes for a Ledgerly release by fetching the open PRs from GitHub and writing a formatted draft. Triggered explicitly with $draft-release-notes <version>."
---

## Steps

1. Use GitHub MCP to list the open PRs in the release-notes-practice repository.
2. For each PR, apply the `pr-label-format` skill to determine which section it belongs to and whether to include it.
3. Group the included PRs under three headings: **Features**, **Fixes**, **Improvements**. Omit any PR the `pr-label-format` skill marks as skip.
4. Draft the release notes for the version passed after the skill mention, in the output format below.
5. Present the draft and ask: "Does this look correct? Should I publish these release notes? (yes / no)"
6. Publish only after an explicit "yes" or "publish". Otherwise, stop.

## Output format

# Release Notes — <version>

## Features
- <one sentence per PR>

## Fixes
- <one sentence per PR>

## Improvements
- <one sentence per PR>
