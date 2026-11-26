````markdown
---
name: go-unit-test-style
description: Apply when writing, editing, or reviewing Go unit tests.
---

1. **Required — Naming.** Use `func TestXxx(t *testing.T)` and name each test after the behavior it verifies.
2. **Critical — Assertions.** Check the error before using the result. Stop the test on an unexpected error. Compare the actual result with the expected result.
3. **Critical — Isolation.** Keep each test independent of shared state, real external services, and test execution order. Use a local `httptest.Server`, in-package fakes, or `t.TempDir()`. Register resource cleanup with `t.Cleanup`.

Doc comments are optional for test functions and unexported test helpers. In both examples, `GetHealthStatus` is a function in the package under test.

## Compliant example

```go
package health

import (
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestGetHealthStatusReturnsResponseCode(t *testing.T) {
	server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, _ *http.Request) {
		w.WriteHeader(http.StatusNoContent)
	}))
	t.Cleanup(server.Close)

	got, err := GetHealthStatus(server.URL)
	if err != nil {
		t.Fatalf("GetHealthStatus: %v", err)
	}
	if want := http.StatusNoContent; got != want {
		t.Errorf("status = %d, want %d", got, want)
	}
}
```

## Noncompliant example

```go
package health

import "testing"

func TestHealth(t *testing.T) {
	status, _ := GetHealthStatus("https://api.example.com/health")
	t.Log(status)
}
```
````
