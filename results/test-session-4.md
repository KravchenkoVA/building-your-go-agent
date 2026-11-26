# Practice 4 — actual session evidence

Tool/model/version and date: Codex CLI (app-server), `gpt-6.2-sol`, `0.171.0-alpha.13`, 2026-11-26 (`medium` reasoning).

Target, mode and head SHA when live: `nebius-academy-templates/go-pr-review-practice` PR #3, `github`, `72d19b992b7af937438327676e17a709623331bd`.

## Recorded conversation and tool events

The session was driven by the two test inputs specified in the exercise. The dialogue and reviewer responses are copied from the live session. Credential values and local workstation paths are masked. Tool events retain the review-relevant inputs and results.

### User

```text
$review-pr 3
```

### Codex · commentary

```text
I’ll use the `review-pr` skill to run case 3 and prepare the review for your confirmation.
```

**Codex → local command (exit 0)**

```sh
/bin/zsh -lc 'cat .agents/skills/review-pr/SKILL.md skills/review-pr.md'
```

**Codex → local command (exit 0)**

```sh
/bin/zsh -lc 'cat review-target.json'
```

**GitHub MCP → `pull_request_read`**

```json
{
  "owner": "nebius-academy-templates",
  "repo": "go-pr-review-practice",
  "pullNumber": 3,
  "method": "get_files",
  "perPage": 10,
  "page": 1
}
```

**GitHub MCP → `pull_request_read`**

```json
{
  "owner": "nebius-academy-templates",
  "repo": "go-pr-review-practice",
  "pullNumber": 3,
  "method": "get"
}
```

**GitHub MCP → `pull_request_read`**

```json
{
  "owner": "nebius-academy-templates",
  "repo": "go-pr-review-practice",
  "pullNumber": 3,
  "method": "get_diff"
}
```

**Codex → local command (exit 0)**

```sh
/bin/zsh -lc "rg --files --hidden -g '"'!'"\\.git' -g '*.md' -g '*reviewer.toml'"
```

**GitHub MCP ← `get_files`**

```json
[
  {
    "filename": "internal/payment/client.go",
    "status": "added",
    "additions": 11,
    "changes": 11,
    "patch": "@@ -0,0 +1,11 @@\n+package payment\n+\n+const apiKey = \"<redacted>\"\n+\n+// Client supplies authorization for the payment adapter.\n+type Client struct{}\n+\n+// Authorization returns the bearer credential used by outgoing requests.\n+func (c Client) Authorization() string {\n+\treturn \"Bearer \" + apiKey\n+}"
  }
]
```

**GitHub MCP ← `get`**

```json
{
  "number": 3,
  "title": "Add payment authorization client",
  "html_url": "https://github.com/nebius-academy-templates/go-pr-review-practice/pull/3",
  "head": {
    "ref": "pr-03",
    "sha": "72d19b992b7af937438327676e17a709623331bd",
    "repo": {
      "full_name": "nebius-academy-templates/go-pr-review-practice"
    }
  },
  "changed_files": 1
}
```

**GitHub MCP ← `get_diff`**

```diff
diff --git a/internal/payment/client.go b/internal/payment/client.go
new file mode 100644
index 0000000..72ec59e
--- /dev/null
+++ b/internal/payment/client.go
@@ -0,0 +1,11 @@
+package payment
+
+const apiKey = "<redacted>"
+
+// Client supplies authorization for the payment adapter.
+type Client struct{}
+
+// Authorization returns the bearer credential used by outgoing requests.
+func (c Client) Authorization() string {
+	return "Bearer " + apiKey
+}
```

**GitHub MCP → `pull_request_read`**

```json
{
  "owner": "nebius-academy-templates",
  "repo": "go-pr-review-practice",
  "pullNumber": 3,
  "method": "get"
}
```

**Codex → local command (exit 0)**

```sh
/bin/zsh -lc 'cat .codex/agents/{style,security,architecture}-reviewer.toml fixture-policy.md'
```

**GitHub MCP ← `get`**

```json
{
  "number": 3,
  "title": "Add payment authorization client",
  "html_url": "https://github.com/nebius-academy-templates/go-pr-review-practice/pull/3",
  "head": {
    "ref": "pr-03",
    "sha": "72d19b992b7af937438327676e17a709623331bd",
    "repo": {
      "full_name": "nebius-academy-templates/go-pr-review-practice"
    }
  },
  "changed_files": 1
}
```

**Codex subagent runtime → style-reviewer started**

`/root/style_retry` · `01a218a7-0704-75f1-abdb-dc63c2184a43` · 2026-11-26T17:18:49.286+01:00

### style-reviewer · commentary

```text
I’ll read the style reviewer instructions before checking the supplied Go diff.
```

**style-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc 'cat reviewers/style-reviewer.md'
```

### style-reviewer · commentary

```text
I’ll apply the application-conventions skill to the changed Go file.
```

**style-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc 'cat skills/app-conventions.md'
```

**style-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc 'cat .agents/skills/app-conventions/SKILL.md'
```

### style-reviewer · final_answer

```text
No style findings.
```

**Codex subagent runtime → style-reviewer completed**

`/root/style_retry` · `01a218a7-0704-75f1-abdb-dc63c2184a43` · 2026-11-26T17:19:00.512+01:00

**Codex collaboration tool ← `wait` (completed)**

**Codex subagent runtime → security-reviewer started**

`/root/security` · `01a218a7-4da9-7063-bdd1-9ed06d75174d` · 2026-11-26T17:19:07.377+01:00

**security-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc 'cat reviewers/security-reviewer.md'
```

**security-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc 'cat fixture-policy.md'
```

### security-reviewer · final_answer

```text
[HIGH] `internal/payment/client.go:3` embeds `const apiKey = "<redacted>"` in source. `Authorization()` uses it as the outgoing bearer credential on line 10. This violates the hardcoded-credentials check. A live credential implemented this way would be exposed to repository readers and would require a rebuild and redeploy to rotate. Inject the credential from runtime secret configuration when constructing `Client`. The fixture policy assigns HIGH severity to this synthetic credential.
```

**Codex collaboration tool ← `wait` (completed)**

**Codex subagent runtime → security-reviewer completed**

`/root/security` · `01a218a7-4da9-7063-bdd1-9ed06d75174d` · 2026-11-26T17:19:19.025+01:00

**Codex collaboration tool ← `wait` (completed)**

**Codex subagent runtime → architecture-reviewer started**

`/root/architecture` · `01a218a7-a052-79b3-b2ba-d5132056e050` · 2026-11-26T17:19:28.527+01:00

### Codex · commentary

```text
The style and security reviews are complete. The architecture reviewer is checking the payment client against the repository’s configuration and dependency injection guidelines.
```

### architecture-reviewer · commentary

```text
I’ll read the architecture reviewer instructions and apply the required skills to the supplied diff.
```

**architecture-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc 'cat reviewers/architecture-reviewer.md'
```

**architecture-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc 'cat skills/architecture-guidelines.md'
```

**architecture-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc "git status --short && rg --files --hidden -g '*ADR*' -g '*adr*' -g 'fixture-policy.md' -g 'AGENTS.md' -g 'SKILL.md' -g '"'!**/.git/**'"'"
```

**architecture-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc 'cat .agents/skills/architecture-guidelines/SKILL.md .agents/skills/app-conventions/SKILL.md'
```

**architecture-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc 'cat docs/adr/ADR-001-layered-architecture.md docs/adr/ADR-002-configuration.md docs/adr/ADR-003-repository-pattern.md docs/adr/ADR-004-dependency-injection.md fixture-policy.md'
```

**architecture-reviewer → local command (exit 0)**

```sh
/bin/zsh -lc 'cat skills/app-conventions.md'
```

### architecture-reviewer · final_answer

```text
No architecture findings.
```

**Codex subagent runtime → architecture-reviewer completed**

`/root/architecture` · `01a218a7-a052-79b3-b2ba-d5132056e050` · 2026-11-26T17:19:45.693+01:00

**Codex collaboration tool ← `wait` (completed)**

### Codex · final_answer

```text
Proposed comment for [nebius-academy-templates/go-pr-review-practice #3](https://github.com/nebius-academy-templates/go-pr-review-practice/pull/3), reviewed at `72d19b9`.

## internal/payment/client.go

- [HIGH] Line 3 hardcodes `const apiKey = "<redacted>"`, and line 10 uses it as the outgoing bearer credential. A real key stored this way would be exposed through repository history and require a rebuild to rotate. Load the key from runtime secret configuration and pass it to `Client` at construction. The repository’s fixture policy assigns HIGH severity to this synthetic credential.

## Summary

HIGH 1 · MEDIUM 0 · LOW 0. Highest severity is HIGH.

Reply `yes` or `post` to publish this comment to PR #3, or `no` to decline.
```

### User

```text
no
```

### Codex · final_answer

```text
The review for PR #3 stayed unpublished.
```

## Delegation path

Codex used the fallback delegation path. The parent read `developer_instructions` from `.codex/agents/{style,security,architecture}-reviewer.toml` and handed the corresponding instructions to each fresh child agent. Each child loaded its shared reviewer file. The runtime events above show each child starting and finishing before the next one started.
