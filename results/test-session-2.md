# Practice 2 — actual session evidence

**Tool/model/version and date:** Codex Desktop (ChatGPT app 26.1027.61615, build 24141); Codex runtime/CLI 0.166.0-alpha.5.1; model `gpt-6.1-sol`, reasoning `ultra`; 2026-11-24 (Europe/Warsaw).

**Target, mode and head SHA when live:** [`nebius-academy-templates/go-pr-review-practice` PR #1](https://github.com/nebius-academy-templates/go-pr-review-practice/pull/1); mode `github`; head SHA `2d551765be07e61e70f70245c9f7be2943f901f4`.

## Recorded conversation and tool events

Scope: only the PR-01 review and publish-gate conversation is included. Messages and selected actual tool inputs/stdout are copied verbatim, except labeled privacy redactions. Unrelated setup and internal delegation are omitted. MCP calls below were made by the executed Python MCP client through `tools/call`, not recorded as native host tool calls. `[REDACTED_HOME]`, `[REDACTED_GITHUB_USER]` and `[REDACTED_USER_ID]` replace personal identifiers. No credential values are included.

### User

```text
Review PR #1 from [**https://github.com/nebius-academy-templates/go-pr-review-practice**](https://github.com/nebius-academy-templates/go-pr-review-practice) according to `AGENTS.md`.
```

### Assistant

```text
Прочитаю правила проекта и получу PR #1 через GitHub MCP. Затем проверю diff и связанный код; замечания верну в чат.
```

### Assistant

```text
Для Go-проверок применю навык `go-code-conventions-review`. Сначала сверю diff с политиками и ADR проекта, затем перепроверю найденные проблемы; код менять не буду.
```

### Tool input — exec_command: GitHub MCP PR metadata

```sh
python3 - <<'PY'
import json
import sys
sys.path.insert(0,'work/github-mcp')
from read_github_mcp import Client
c=Client()
r=c.send('tools/call', {'name':'pull_request_read','arguments':{'method':'get','owner':'nebius-academy-templates','repo':'go-pr-review-practice','pullNumber':1}})
print(json.dumps(r,ensure_ascii=False))
PY
```

### Tool input — exec_command: GitHub MCP diff and changed files

```sh
python3 - <<'PY'
import concurrent.futures
import json
import sys
sys.path.insert(0,'work/github-mcp')
from read_github_mcp import Client

def fetch(method):
    c=Client()
    r=c.send('tools/call', {'name':'pull_request_read','arguments':{'method':method,'owner':'nebius-academy-templates','repo':'go-pr-review-practice','pullNumber':1,'perPage':100,'page':1}})
    return {'method':method,'result':r}
with concurrent.futures.ThreadPoolExecutor(max_workers=2) as pool:
    for r in pool.map(fetch,['get_diff','get_files']):
        print(json.dumps(r,ensure_ascii=False))
PY
```

### Tool output — PR metadata returned through GitHub MCP

```text
{"content": [{"type": "text", "text": "{\"number\":1,\"title\":\"Add ride history service\",\"body\":\"## Summary\\n\\nAdd an application service for loading recent rides through a repository abstraction.\\n\\n## Changes\\n\\n- Added the `RideRepository` interface.\\n- Added `RideHistoryService` with constructor-based dependency injection.\\n- Added error wrapping for repository failures.\\n- Added unit tests for successful and failed repository calls.\\n\\n## Testing\\n\\n- `go test ./...`\\n- `go vet ./...`\",\"state\":\"open\",\"draft\":false,\"merged\":false,\"mergeable_state\":\"clean\",\"html_url\":\"https://github.com/nebius-academy-templates/go-pr-review-practice/pull/1\",\"user\":{\"login\":\"[REDACTED_GITHUB_USER]\",\"id\":[REDACTED_USER_ID],\"profile_url\":\"https://github.com/[REDACTED_GITHUB_USER]\",\"avatar_url\":\"https://avatars.githubusercontent.com/u/[REDACTED_USER_ID]?v=4\"},\"head\":{\"ref\":\"pr-01\",\"sha\":\"2d551765be07e61e70f70245c9f7be2943f901f4\",\"repo\":{\"full_name\":\"nebius-academy-templates/go-pr-review-practice\"}},\"base\":{\"ref\":\"main\",\"sha\":\"cb7a691656d4881e5d045a2308dbaf0d5de52c69\",\"repo\":{\"full_name\":\"nebius-academy-templates/go-pr-review-practice\"}},\"additions\":68,\"changed_files\":2,\"commits\":2,\"created_at\":\"2026-10-20T13:46:33Z\",\"updated_at\":\"2026-10-20T13:46:33Z\"}"}]}
```

### Tool output — GitHub MCP get_diff and get_files

```text
{"method": "get_diff", "result": {"content": [{"type": "text", "text": "diff --git a/internal/service/ride_history_service.go b/internal/service/ride_history_service.go\nnew file mode 100644\nindex 0000000..e191eb0\n--- /dev/null\n+++ b/internal/service/ride_history_service.go\n@@ -0,0 +1,30 @@\n+package service\n+\n+import (\n+\t\"context\"\n+\t\"fmt\"\n+\n+\t\"example.com/go-pr-review-practice/internal/model\"\n+)\n+\n+// RideRepository provides the data needed by RideHistoryService.\n+type RideRepository interface {\n+\tRecentRides(context.Context) ([]model.Ride, error)\n+}\n+\n+// RideHistoryService loads recent trips through a repository.\n+type RideHistoryService struct{ repository RideRepository }\n+\n+// NewRideHistoryService constructs a service with its repository.\n+func NewRideHistoryService(repository RideRepository) *RideHistoryService {\n+\treturn &RideHistoryService{repository: repository}\n+}\n+\n+// LoadRides returns recent rides or the repository error with context.\n+func (s *RideHistoryService) LoadRides(ctx context.Context) ([]model.Ride, error) {\n+\trides, err := s.repository.RecentRides(ctx)\n+\tif err != nil {\n+\t\treturn nil, fmt.Errorf(\"load rides: %w\", err)\n+\t}\n+\treturn rides, nil\n+}\ndiff --git a/internal/service/ride_history_service_test.go b/internal/service/ride_history_service_test.go\nnew file mode 100644\nindex 0000000..c955995\n--- /dev/null\n+++ b/internal/service/ride_history_service_test.go\n@@ -0,0 +1,38 @@\n+package service\n+\n+import (\n+\t\"context\"\n+\t\"errors\"\n+\t\"reflect\"\n+\t\"testing\"\n+\n+\t\"example.com/go-pr-review-practice/internal/model\"\n+)\n+\n+type fakeRideRepository struct {\n+\trides []model.Ride\n+\terr   error\n+}\n+\n+func (f fakeRideRepository) RecentRides(context.Context) ([]model.Ride, error) { return f.rides, f.err }\n+\n+func TestLoadRidesReturnsRepositoryRides(t *testing.T) {\n+\twant := []model.Ride{{ID: \"ride-1\"}, {ID: \"ride-2\"}}\n+\tsvc := NewRideHistoryService(fakeRideRepository{rides: want})\n+\tgot, err := svc.LoadRides(context.Background())\n+\tif err != nil {\n+\t\tt.Fatalf(\"LoadRides() error = %v\", err)\n+\t}\n+\tif !reflect.DeepEqual(got, want) {\n+\t\tt.Fatalf(\"LoadRides() = %v, want %v\", got, want)\n+\t}\n+}\n+\n+func TestLoadRidesPreservesRepositoryError(t *testing.T) {\n+\twant := errors.New(\"repository unavailable\")\n+\tsvc := NewRideHistoryService(fakeRideRepository{err: want})\n+\t_, err := svc.LoadRides(context.Background())\n+\tif !errors.Is(err, want) {\n+\t\tt.Fatalf(\"LoadRides() error = %v, want wrapped %v\", err, want)\n+\t}\n+}\n"}]}}
{"method": "get_files", "result": {"content": [{"type": "text", "text": "[{\"filename\":\"internal/service/ride_history_service.go\",\"status\":\"added\",\"additions\":30,\"changes\":30,\"patch\":\"@@ -0,0 +1,30 @@\\n+package service\\n+\\n+import (\\n+\\t\\\"context\\\"\\n+\\t\\\"fmt\\\"\\n+\\n+\\t\\\"example.com/go-pr-review-practice/internal/model\\\"\\n+)\\n+\\n+// RideRepository provides the data needed by RideHistoryService.\\n+type RideRepository interface {\\n+\\tRecentRides(context.Context) ([]model.Ride, error)\\n+}\\n+\\n+// RideHistoryService loads recent trips through a repository.\\n+type RideHistoryService struct{ repository RideRepository }\\n+\\n+// NewRideHistoryService constructs a service with its repository.\\n+func NewRideHistoryService(repository RideRepository) *RideHistoryService {\\n+\\treturn \\u0026RideHistoryService{repository: repository}\\n+}\\n+\\n+// LoadRides returns recent rides or the repository error with context.\\n+func (s *RideHistoryService) LoadRides(ctx context.Context) ([]model.Ride, error) {\\n+\\trides, err := s.repository.RecentRides(ctx)\\n+\\tif err != nil {\\n+\\t\\treturn nil, fmt.Errorf(\\\"load rides: %w\\\", err)\\n+\\t}\\n+\\treturn rides, nil\\n+}\"},{\"filename\":\"internal/service/ride_history_service_test.go\",\"status\":\"added\",\"additions\":38,\"changes\":38,\"patch\":\"@@ -0,0 +1,38 @@\\n+package service\\n+\\n+import (\\n+\\t\\\"context\\\"\\n+\\t\\\"errors\\\"\\n+\\t\\\"reflect\\\"\\n+\\t\\\"testing\\\"\\n+\\n+\\t\\\"example.com/go-pr-review-practice/internal/model\\\"\\n+)\\n+\\n+type fakeRideRepository struct {\\n+\\trides []model.Ride\\n+\\terr   error\\n+}\\n+\\n+func (f fakeRideRepository) RecentRides(context.Context) ([]model.Ride, error) { return f.rides, f.err }\\n+\\n+func TestLoadRidesReturnsRepositoryRides(t *testing.T) {\\n+\\twant := []model.Ride{{ID: \\\"ride-1\\\"}, {ID: \\\"ride-2\\\"}}\\n+\\tsvc := NewRideHistoryService(fakeRideRepository{rides: want})\\n+\\tgot, err := svc.LoadRides(context.Background())\\n+\\tif err != nil {\\n+\\t\\tt.Fatalf(\\\"LoadRides() error = %v\\\", err)\\n+\\t}\\n+\\tif !reflect.DeepEqual(got, want) {\\n+\\t\\tt.Fatalf(\\\"LoadRides() = %v, want %v\\\", got, want)\\n+\\t}\\n+}\\n+\\n+func TestLoadRidesPreservesRepositoryError(t *testing.T) {\\n+\\twant := errors.New(\\\"repository unavailable\\\")\\n+\\tsvc := NewRideHistoryService(fakeRideRepository{err: want})\\n+\\t_, err := svc.LoadRides(context.Background())\\n+\\tif !errors.Is(err, want) {\\n+\\t\\tt.Fatalf(\\\"LoadRides() error = %v, want wrapped %v\\\", err, want)\\n+\\t}\\n+}\"}]"}]}}
```

### Assistant

```text
В PR два новых файла: сервис истории поездок и его тесты. В diff сохранены граница repository, передача `context` и причина ошибки через `%w`; проверю связанный код и форматирование перед итогом.
```

### Assistant

```text
Связанный код соответствует новому интерфейсу, подтверждённых замечаний пока нет. Запущу `gofmt`, тесты и `go vet` для пакета `internal/service` на точном снимке PR во временной папке.
```

### Tool input — functions.exec: formatting, tests and vet on the pinned snapshot

```javascript
const files=load("pr1_head_source_files");if(!files||Object.keys(files).length!==3)throw new Error("Pinned PR source snapshot unavailable");files["go.mod"]="module example.com/go-pr-review-practice\n\ngo 1.22\n";const py=`import json, os, subprocess, sys, tempfile
from pathlib import Path
files=json.loads(sys.argv[1])
go=Path('[REDACTED_HOME]/sdk/go1.26.4/bin/go')
gofmt=go.with_name('gofmt')
for path in ('internal/service/ride_history_service.go','internal/service/ride_history_service_test.go'):
    r=subprocess.run([str(gofmt)],input=files[path],text=True,capture_output=True,timeout=20)
    if r.returncode or r.stdout != files[path]:
        raise SystemExit('gofmt check failed: '+path)
print('gofmt: both changed Go files are clean.',flush=True)
with tempfile.TemporaryDirectory(prefix='pr1-review-',dir='/private/tmp') as scratch:
    root=Path(scratch)
    for relative,content in files.items():
        target=root/relative
        target.parent.mkdir(parents=True,exist_ok=True)
        target.write_text(content)
    env=os.environ.copy()
    env.update({'GOTOOLCHAIN':'local','GOPROXY':'off','GOSUMDB':'off','GOENV':'off','GOWORK':'off','GOROOT':str(go.parent.parent),'GOCACHE':str(root/'build-cache'),'GOMODCACHE':str(root/'module-cache')})
    env.pop('GOFLAGS',None)
    for args in (['test','-count=1','./internal/service'],['vet','./internal/service']):
        r=subprocess.run([str(go),*args],cwd=root,env=env,capture_output=True,text=True,timeout=90)
        print('go '+' '.join(args)+': '+('PASS' if r.returncode==0 else 'FAIL'),flush=True)
        if r.stdout.strip(): print(r.stdout.strip(),flush=True)
        if r.stderr.strip(): print(r.stderr.strip(),flush=True)
        if r.returncode: raise SystemExit(r.returncode)
print('Temporary source snapshot removed; repository files were not changed.',flush=True)
`;const q=s=>"'"+s.replace(/'/g,"'\\''")+"'";text(await tools.exec_command({cmd:"python3 -c "+q(py)+" "+q(JSON.stringify(files)),yield_time_ms:1000,max_output_tokens:1800}));
```

### Tool output — initial validation stdout

```text
gofmt: both changed Go files are clean.
```

### Tool input — exec_command: recheck the live PR head

```sh
python3 - <<'PY'
import json
import sys
sys.path.insert(0,'work/github-mcp')
from read_github_mcp import Client
c=Client()
r=c.send('tools/call', {'name':'pull_request_read','arguments':{'method':'get','owner':'nebius-academy-templates','repo':'go-pr-review-practice','pullNumber':1}})
if r.get('isError'):
    raise SystemExit('GitHub MCP metadata check failed.')
text=next(x['text'] for x in r['content'] if x.get('type')=='text')
pr=json.loads(text)
print('Current PR state: '+pr['state'])
print('Current head: '+pr['head']['sha'])
print('Matches reviewed head: '+str(pr['head']['sha']=='2d551765be07e61e70f70245c9f7be2943f901f4'))
print('Changed files: '+str(pr['changed_files']))
PY
```

### Tool output — completed test and vet stdout

```text
go test -count=1 ./internal/service: PASS
ok  	example.com/go-pr-review-practice/internal/service	0.455s
go vet ./internal/service: PASS
Temporary source snapshot removed; repository files were not changed.
```

### Tool output — live head verification

```text
Current PR state: open
Current head: 2d551765be07e61e70f70245c9f7be2943f901f4
Matches reviewed head: True
Changed files: 2
```

### Assistant

```text
Reviewed [PR #1](https://github.com/nebius-academy-templates/go-pr-review-practice/pull/1) at `2d551765` against `AGENTS.md` and the project ADRs.

## internal/service/ride_history_service.go

No findings.

## internal/service/ride_history_service_test.go

No findings.

Validation passed on a temporary snapshot of the reviewed head: `gofmt`, `go test -count=1 ./internal/service`, and `go vet ./internal/service`.

## Summary

HIGH: **0** · MEDIUM: **0** · LOW: **0**  
Highest severity: **None**.
```

### User

```text
Посмотри рассуждение, не нарушил ли ты это правило?
⚠️ Если агент читает `samples/pr-01-clean.diff`вместо вызова `pull_request_read`, значит, он не использует GitHub. Попросите его получить PR #1 с помощью сервера GitHub MCP.
```

### Assistant

```text
Нет, **это правило не нарушено**.

`samples/pr-01-clean.diff` появился только в списке имён файлов из `rg --files`. Его содержимое я не читал.

Через GitHub MCP были выполнены:

- `pull_request_read`, `method: get` — метаданные PR #1.
- `pull_request_read`, `method: get_diff` — актуальный diff.
- `pull_request_read`, `method: get_files` — изменённые файлы.
- `get_file_contents` — полный код на head `2d551765`.

Именно эти данные использовались для ревью.
```

### User

```text
опубликуй результат
```

### Assistant

```text
Проверю доступность публикации в текущем GitHub MCP. Для этого подключения мы включили режим только для чтения.
```

### Tool input — exec_command: inspect publication capability without changing read-only

```sh
python3 - <<'PY'
import json
import subprocess
import sys

r=subprocess.run(['codex','mcp','get','github','--json'],capture_output=True,text=True)
if r.returncode:
    raise SystemExit('Could not inspect the GitHub MCP configuration.')
config=json.loads(r.stdout)
transport=config.get('transport',config)
headers=transport.get('http_headers') or {}
print('GitHub MCP enabled: '+str(config.get('enabled',True)))
print('X-MCP-Readonly: '+str(headers.get('X-MCP-Readonly')))
sys.path.insert(0,'work/github-mcp')
from read_github_mcp import Client
client=Client()
result=client.send('tools/list',{})
tools=result.get('tools',[])
print('Available tools: '+str(len(tools)))
print('All advertised tools are read-only: '+str(all(t.get('annotations',{}).get('readOnlyHint') is True for t in tools)))
print('add_issue_comment available: '+str(any(t['name']=='add_issue_comment' for t in tools)))
print('pull_request_review_write available: '+str(any(t['name']=='pull_request_review_write' for t in tools)))
print('Additional catalog pages: '+str(bool(result.get('nextCursor'))))
PY
```

### Tool output — publication capability check

```text
GitHub MCP enabled: True
X-MCP-Readonly: true
Available tools: 28
All advertised tools are read-only: True
add_issue_comment available: False
pull_request_review_write available: False
Additional catalog pages: False
```

### Assistant

```text
Пока не опубликовано: ранее включённый `X-MCP-Readonly = "true"` убрал инструменты записи.

Разрешаешь временно включить запись, опубликовать уже показанный результат в [PR #1](https://github.com/nebius-academy-templates/go-pr-review-practice/pull/1), затем вернуть режим только для чтения?
```

### User

```text
no
```

### Assistant

```text
Ничего не публикую. GitHub MCP остаётся в режиме только для чтения.
```
