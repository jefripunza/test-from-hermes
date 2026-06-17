---
name: golang
description: "Go expert: REST API, Clean/Hexagonal Architecture, DDD, CQRS, Event Driven, Dependency Injection, concurrency, middleware, repository pattern. Production-grade backend."
metadata:
  hermes:
    tags: [go, golang, backend, rest-api, clean-architecture, ddd, cqrs, concurrency]
---

# Go — Senior Full-Stack Engineer

Expert in Go backend development. Build production-grade services.

## Expertise

- Go (Golang)
- REST API design & implementation
- Clean Architecture / Hexagonal Architecture
- Domain-Driven Design (DDD)
- CQRS
- Event-Driven Architecture
- Dependency Injection
- Concurrency (goroutines, channels, errgroup)
- Middleware (logging, auth, rate-limit, recovery)
- Repository Pattern
- Context propagation
- Structured logging
- Input validation

## Mandatory Code Quality Gate

Code is NOT complete until passing:

```bash
gofmt ./...
goimports ./...
golangci-lint run
gosec ./...
go test ./...
go test -race ./...
```

Requirements:
- `golangci-lint` = 0 issues
- `gosec` = 0 High severity findings
- `gofmt` compliant
- `goimports` compliant
- Race detector pass
- Unit tests pass

## Avoid

- Unused variables / imports
- Unchecked errors
- Dead code
- Naked returns
- Magic numbers
- Duplicated logic
- Goroutine leaks
- Race conditions

## Architecture (Clean / Hexagonal)

```
project/
├── cmd/              # Entry points (main.go)
│   └── server/
├── internal/
│   ├── domain/       # Entities, value objects, domain errors
│   ├── application/  # Use cases / services
│   ├── ports/        # Interfaces (repositories, presenters)
│   └── adapters/     # Implementations (DB, HTTP, cache)
│       ├── handlers/ # HTTP handlers
│       ├── repo/     # Repository impl (PostgreSQL, Redis)
│       └── middleware/
├── pkg/              # Public shared utilities
├── api/              # OpenAPI specs
├── config/           # Config loading
└── migrations/       # SQL migrations
```

## HTTP Handler Pattern

```go
// internal/adapters/handlers/user.go
func (h *UserHandler) Create(w http.ResponseWriter, r *http.Request) {
    var req dto.CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        respondError(w, http.StatusBadRequest, "invalid request body")
        return
    }
    if err := h.validator.Struct(req); err != nil {
        respondError(w, http.StatusUnprocessableEntity, err.Error())
        return
    }
    ctx := r.Context()
    user, err := h.svc.Create(ctx, req)
    if err != nil {
        respondError(w, http.StatusInternalServerError, err.Error())
        return
    }
    respondJSON(w, http.StatusCreated, user)
}
```

## Concurrency — Error Group

```go
g, ctx := errgroup.WithContext(ctx)
g.Go(func() error { return fetchUsers(ctx) })
g.Go(func() error { return fetchPosts(ctx) })
if err := g.Wait(); err != nil {
    return fmt.Errorf("concurrent fetch: %w", err)
}
```

## Repository Pattern

```go
type UserRepository interface {
    Create(ctx context.Context, u *domain.User) error
    FindByID(ctx context.Context, id uuid.UUID) (*domain.User, error)
    List(ctx context.Context, filter domain.Filter) ([]domain.User, error)
}
```

## Before Finalizing Any Go Code

1. Verify `gofmt` compliance
2. Verify `goimports` compliance
3. Verify `golangci-lint` compliance
4. Verify `gosec` compliance
5. Verify race-condition safety
6. Verify context propagation
7. Verify proper error handling (wrapping)

## Verification

```bash
go vet ./...
go test -race ./...
go test -bench=. -benchmem ./...
go build -o /dev/null .
```
