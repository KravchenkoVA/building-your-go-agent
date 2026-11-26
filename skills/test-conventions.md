## Rules

1. **[LOW] Naming.** Use `func TestXxx(t *testing.T)` and name each test after the behavior it verifies.
2. **[MEDIUM] Assertions.** Check the error before using the result. Stop the test on an unexpected error. Compare the observable result with the expected result and report both on failure.
3. **[MEDIUM] Isolation.** Keep each test independent of shared state, real external services, and test execution order. Use a local `httptest.Server`, in-package fakes, or `t.TempDir()`. Register resource cleanup with `t.Cleanup`.

Doc comments are optional for test functions and unexported test helpers. In both examples, `GetHealthStatus` is a function in the package under test.

## Template

### Compliant example

```go
package health

import (
	"fmt"
	"net/http"
	"testing"
)

// GetHealthStatus returns the HTTP status of the health endpoint.
func GetHealthStatus(client *http.Client, endpoint string) (int, error) {
	response, err := client.Get(endpoint)
	if err != nil {
		return 0, fmt.Errorf("get health status: %w", err)
	}
	if err := response.Body.Close(); err != nil {
		return 0, fmt.Errorf("close health response: %w", err)
	}
	return response.StatusCode, nil
}

type fakeHealthTransport struct {
	statusCode int
}

func (transport fakeHealthTransport) RoundTrip(*http.Request) (*http.Response, error) {
	return &http.Response{StatusCode: transport.statusCode, Body: http.NoBody}, nil
}

func TestGetHealthStatusReturnsResponseCode(t *testing.T) {
	want := http.StatusNoContent
	client := &http.Client{Transport: fakeHealthTransport{statusCode: want}}

	got, err := GetHealthStatus(client, "https://health.test/health")
	if err != nil {
		t.Fatalf("GetHealthStatus: %v", err)
	}
	if got != want {
		t.Errorf("status = %d, want %d", got, want)
	}
}
```

### Non-compliant example

```go
package health

import (
	"net/http"
	"testing"
)

func TestHealth(t *testing.T) {
	status, _ := GetHealthStatus(http.DefaultClient, "https://api.example.com/health")
	t.Log(status)
}
```
