---
title: Testing with Fakes
type: concept
tags: [testing, fakes, mocks, dependency-injection, interfaces, functions]
created: 2026-04-11
updated: 2026-04-11
sources: [raw/fakes-in-golang.md]
---

Go has no dynamic mocking framework. Test doubles are created via dependency injection — interfaces or function types. This forces explicit, auditable dependencies. See also: [[Testing]], [[Structs and Interfaces]].

## Why no mocks?

Go cannot dynamically intercept package-level function calls at runtime (no monkey-patching without `unsafe`). The design intent is: **make dependencies explicit in the type system**, then swap them in tests.

This mirrors the Java pattern of injecting `Supplier` / functional interfaces — just lighter.

## Pattern 1: Interface injection (recommended for complex dependencies)

```go
// Define a minimal interface for the dependency
type Randomizer interface {
    IntN(n int) int
}

// Production implementation wraps the real package
type defaultRandomizer struct{}

func (defaultRandomizer) IntN(n int) int {
    return rand.IntN(n) // math/rand/v2 (Go 1.22+)
}

// Service accepts the interface
type MyService struct {
    rng Randomizer
}

func NewMyService(rng Randomizer) *MyService {
    if rng == nil {
        rng = defaultRandomizer{}
    }
    return &MyService{rng: rng}
}

func (s *MyService) PickItem(items []string) string {
    return items[s.rng.IntN(len(items))]
}
```

```go
// Test: hand-written fake
type fakeRandomizer struct {
    nextIntN func(n int) int
}

func (f fakeRandomizer) IntN(n int) int { return f.nextIntN(n) }

func TestPickItem(t *testing.T) {
    fake := fakeRandomizer{nextIntN: func(n int) int { return 0 }} // always first
    svc := NewMyService(fake)

    got := svc.PickItem([]string{"a", "b", "c"})
    if got != "a" {
        t.Errorf("got %q, want %q", got, "a")
    }
}
```

## Pattern 2: Function type injection (best for single-method cases)

Direct analog of Java's `IntUnaryOperator`. Lighter than defining an interface.

```go
type GameService struct {
    randIntN func(int) int
}

func NewGameService(r func(int) int) *GameService {
    if r == nil {
        r = rand.IntN // math/rand/v2
    }
    return &GameService{randIntN: r}
}

func (s *GameService) Roll(sides int) int {
    return s.randIntN(sides) + 1 // 1-indexed
}
```

```go
// Test: inline function — no fake struct needed
func TestRoll(t *testing.T) {
    svc := NewGameService(func(n int) int { return 2 }) // always 3 (2+1)
    if got := svc.Roll(6); got != 3 {
        t.Errorf("got %d, want 3", got)
    }
}
```

## Pattern 3: Seeded rand (no architecture change)

When you don't want to change the design but need determinism:

```go
// math/rand/v2 — deterministic with fixed seed
r := rand.New(rand.NewPCG(42, 123))
value := r.IntN(100) // reproducible sequence

// Use in tests where distribution matters, not specific values
// Or for property-based testing / fuzzing
```

## Apply the same pattern to other dependencies

The interface/function injection pattern applies universally:

```go
// Clock injection — avoid time.Now() in business logic
type Clock interface {
    Now() time.Time
}

type realClock struct{}
func (realClock) Now() time.Time { return time.Now() }

// For tests:
type fixedClock struct{ t time.Time }
func (c fixedClock) Now() time.Time { return c.t }

// HTTP client injection
type HTTPDoer interface {
    Do(req *http.Request) (*http.Response, error)
}
// *http.Client satisfies this already — no wrapper needed

// File system injection (Go 1.16+ fs.FS)
type Processor struct {
    fs fs.FS
}
// In tests: os.DirFS("testdata") or fstest.MapFS{}
```

## Avoid these approaches

```go
// BAD: global variable swap — breaks with -race flag
var randIntn = rand.Intn // global
func testHook() { randIntn = func(n int) int { return 42 } }

// BAD: init() registration — hidden coupling, hard to test
func init() { globalRNG = productionRNG }

// BAD: reflect/unsafe for monkey-patching — unsupported, breaks on optimizations
```

## Comparison: Go vs Java/Kotlin

| Concern | Java/Kotlin | Go |
|---------|-------------|-----|
| Inject random | `Supplier<Integer>` / `IntUnaryOperator` | `func(int) int` or interface |
| Mock framework | Mockito, MockK | None (hand-write fakes) |
| Interface for single method | Functional interface | `func` type (preferred) |
| Multi-method mock | `@Mock` | Hand-write struct or use `mockery` |
| Clock | `Clock` interface (JSR-310) | `func() time.Time` or `Clock` interface |

## Generated mocks (when hand-writing is too much)

```bash
# mockery — generates mocks from interfaces
go install github.com/vektra/mockery/v2@latest
mockery --name=UserRepo --output=./mocks

# Usage in tests
mock := mocks.NewUserRepo(t)
mock.On("GetUser", ctx, 1).Return(&User{ID: 1}, nil)
```
