## Rules

1. **[LOW] Naming.** Use `MixedCaps` for exported names and `mixedCaps` for unexported names. Follow Go capitalization for initialisms such as `ID`, `URL`, `HTTP`, and `API`. Examples include `UserID`, `apiURL`, and `httpClient`.
2. **[LOW] Comments and formatting.** Document each exported declaration with a comment that starts with its name. Format Go code with `gofmt`.
3. **[MEDIUM] Errors.** Explicitly handle or return every error. When adding context, preserve the original error with `fmt.Errorf("…: %w", err)`.

## Template

### Compliant example

```go
package config

import (
	"fmt"
	"os"
)

// ReadAPIConfig reads the API configuration from a file.
func ReadAPIConfig(path string) ([]byte, error) {
	data, err := os.ReadFile(path)
	if err != nil {
		return nil, fmt.Errorf("read API config: %w", err)
	}
	return data, nil
}
```

### Non-compliant example

```go
package config

import "os"

func ReadApiConfig(file_path string) []byte{
data,_:=os.ReadFile(file_path)
return data
}
```
