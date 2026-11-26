---
name: pr-label-format
description: Maps GitHub pull request labels to release notes sections when preparing a draft from PRs.
---

# PR labels for release notes

| GitHub label | Section |
| --- | --- |
| `feature` | Features |
| `fix` | Fixes |
| `improvement` | Improvements |
| `chore` | Skip |
| `internal` | Skip |

Skip a PR carrying `chore` or `internal`, even if it also has a release label.
If a PR has zero or multiple matching release labels, flag it for review and
keep it out of the draft until its category is clear.

For features, name the new user capability. For fixes, describe the corrected
behavior. For improvements, describe the observable benefit.
