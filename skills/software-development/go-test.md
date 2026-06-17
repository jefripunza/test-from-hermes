---
name: go-test
description: "Go testing expert: table-driven tests, integration tests, benchmarks, mocking. Min 80% coverage, preferred 90%+."
metadata:
  hermes:
    tags: [go, golang, testing, table-driven, benchmark, coverage]
---

# Go Test — Testing in Go

Expert in Go testing patterns. Table-driven tests, integration, benchmarks.

## Expertise

- Go `testing` package
- Table-driven tests
- Subtests (`t.Run`)
- Integration tests (testcontainers-go)
- Benchmarks
- Fuzz testing
- Mocking strategies (interfaces, mockgen)
- Test helpers & fixtures
- Coverage analysis

## Coverage Targets

| Area | Min | Preferred |
|------|-----|-----------|
| Overall | 80% | 90%+ |
| Domain logic | 90% | 95%+ |
| Handlers | 80% | 90%+ |
| Repositories | 75% | 85%+ |
| Integration | key paths | critical flows |

## Table-Driven Tests

```go
func TestCalculateTotal(t *testing.T) {
    t.Parallel()
    tests := []struct {
        name     string
        items    []Item
        discount float64
        want     float64
        wantErr  bool
    }{
        {name: "empty cart", items: []Item{}, discount: 0, want: 0, wantErr: false},
        {name: "single item", items: []Item{{Price: 10}}, discount: 0, want: 10, wantErr: false},
        {name: "with discount", items: []Item{{Price: 100}}, discount: 0.1, want: 90, wantErr: false},
        {name: "invalid discount", items: []Item{{Price: 10}}, discount: 1.5, want: 0, wantErr: true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := CalculateTotal(tt.items, tt.discount)
            if (err != nil) != tt.wantErr {
                t.Errorf("error = %v, wantErr = %v", err, tt.wantErr)
            }
            if got != tt.want {
                t.Errorf("got %v, want %v", got, tt.want)
            }
        })
    }
}
```

## Handler Test

```go
func TestUserHandler_Create(t *testing.T) {
    t.Parallel()
    req := httptest.NewRequest("POST", "/users", body)
    rec := httptest.NewRecorder()
    handler.Create(rec, req)
    assert.Equal(t, http.StatusCreated, rec.Code)
}
```

## Benchmark

```go
func BenchmarkParse(b *testing.B) {
    for b.Loop() {
        Parse("input data here")
    }
}
```

## Requirements

- Edge case coverage (empty, null, boundary)
- Error coverage (all error paths)
- Mocking strategy (interfaces, not concrete types)
- Maintainable test structure (helpers, fixtures)
- Parallel subtests where possible (`t.Parallel()`)
