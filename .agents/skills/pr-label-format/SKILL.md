---
name: pr-label-format
description: "PR label conventions for release notes. Load only when writing or drafting release notes from a list of pull requests — maps GitHub labels to sections (feature → Features, fix → Fixes, improvement → Improvements) and names the labels to skip entirely (chore, internal). Do not load for code review, file analysis, or any task that does not involve processing a pull request list."
---

## Label → category mapping

| Label | Release notes section | Include? |
|---|---|---|
| `feature` | Features | ✅ yes |
| `fix` | Fixes | ✅ yes |
| `improvement` | Improvements | ✅ yes |
| `chore` | — | ❌ skip |
| `internal` | — | ❌ skip |

## Writing rules per category

**Features**: describe what the user can now do that they could not do before. Start with the new capability, written from the user's point of view.

**Fixes**: describe what was broken and is now correct. Start with "Fixed" or the noun of what was fixed. One sentence — do not explain the root cause.

**Improvements**: describe the observable improvement — speed, clarity, accessibility. Avoid implementation details. Write what the user notices, not what changed in the code.
