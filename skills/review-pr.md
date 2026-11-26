## Steps

1. Read the case number from the request and find the corresponding PR in `review-target.json`.
2. Fetch the PR and its diff with GitHub MCP `pull_request_read` using `get` and `get_diff`.
3. Review only changed files within the requested scope.
4. Load `app-conventions` for `.go` files that do not end in `_test.go` and `test-conventions` for `_test.go` files. Load each skill only if its file type is in scope.
5. Apply the security rules in `AGENTS.md`.
6. Format the findings and summary as specified in `AGENTS.md`.
7. Show the full review and its posting target, then ask for confirmation.
8. Post with `add_issue_comment` only after a subsequent explicit `yes` or `post`. If the user replies `no`, acknowledge the decision and stop. Never approve, merge, or close PRs.
