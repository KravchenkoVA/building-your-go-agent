---
name: go-code-style
description: Apply when writing, editing, or reviewing Go code, including Go examples in documentation.
---

## Rules

1. **Required — Naming.** Use `MixedCaps` for exported names and `mixedCaps` for unexported names. Follow Go capitalization for initialisms such as `ID`, `URL`, `HTTP`, and `API`. Examples include `UserID`, `apiURL`, and `httpClient`.
2. **Required — Comments and formatting.** Document each exported declaration with a comment that starts with its name. Format Go code and examples with `gofmt`.
3. **Critical — Errors.** Explicitly handle or return every error. When adding context, preserve the original error with `fmt.Errorf("…: %w", err)`.

## Compliant example

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

## Noncompliant example

```go
package config

import "os"

func ReadApiConfig(file_path string) []byte{
data,_:=os.ReadFile(file_path)
return data
}
```
