# Team architecture guidelines

These four accepted decisions in `docs/adr/` define the team's architecture.

## ADR-001 — Layered Architecture

Handlers and `cmd/` entrypoints parse inputs and format outputs. Services own business logic. Entrypoints call services and have no direct access to persistence.

## ADR-002 — Configuration

Load runtime configuration and secrets centrally in `internal/config` or at the composition root. Pass those values through constructors. Services use the injected configuration and avoid ad hoc `os.Getenv` calls or embedded credentials. Report embedded credentials once as a security finding, with this configuration rule as part of the remediation.

## ADR-003 — Repository Pattern

Services obtain data through small repository interfaces, usually defined near the consumer. Only the concrete adapters in `internal/repository` may depend on `internal/api` or `database/sql`. Injecting a concrete API client satisfies dependency injection but still bypasses the repository layer. A repository bypass is ordinarily `[MEDIUM]`. Tests may wire repository adapters to a local `httptest.Server`.

## ADR-004 — Dependency Injection

Pass dependencies through `NewXxx` constructors or function parameters. Avoid global services, global clients, and service locators. Tests may supply fakes or local adapters.
