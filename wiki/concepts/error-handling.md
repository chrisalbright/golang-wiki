---
title: Error Handling
type: concept
tags: [errors, panic, recover, defer, sentinel-errors]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Errors in Go are values — returned, not thrown. This keeps control flow explicit and readable. See also: [[Language Basics]], [[Structs and Interfaces]].

## The error interface

```go
// error is a built-in interface
type error interface {
    Error() string
}

// Convention: return (result, error), check immediately
result, err := doSomething()
if err != nil {
    return fmt.Errorf("doSomething: %w", err) // wrap with context
}
```

## Creating errors

```go
// Simple
errors.New("something went wrong")

// Formatted
fmt.Errorf("user %d not found", id)

// Custom error type (adds structured data)
type NotFoundError struct {
    Resource string
    ID       int
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s with id %d not found", e.Resource, e.ID)
}

return &NotFoundError{Resource: "user", ID: 42}

// errors.Join — combine multiple errors (Go 1.20+)
err := errors.Join(err1, err2, err3)
```

## Wrapping and unwrapping

```go
// Wrap preserves the original error in the chain
err := fmt.Errorf("database query: %w", originalErr)

// errors.Is — checks anywhere in the chain
var ErrNotFound = errors.New("not found") // sentinel

if errors.Is(err, ErrNotFound) { ... }

// errors.As — extract typed error from chain
var nfe *NotFoundError
if errors.As(err, &nfe) {
    fmt.Println("resource:", nfe.Resource)
}

// Unwrap manually
unwrapped := errors.Unwrap(err)
```

## Sentinel errors

```go
// Package-level sentinel errors
var (
    ErrNotFound   = errors.New("not found")
    ErrForbidden  = errors.New("forbidden")
    ErrConflict   = errors.New("conflict")
)

// Callers check with errors.Is, not ==
if errors.Is(err, ErrNotFound) {
    // handle 404
}
```

## Error handling patterns

```go
// Pattern 1: wrap and return (most common)
func getUser(id int) (*User, error) {
    u, err := db.QueryUser(id)
    if err != nil {
        return nil, fmt.Errorf("getUser %d: %w", id, err)
    }
    return u, nil
}

// Pattern 2: handle and continue
for _, item := range items {
    if err := process(item); err != nil {
        log.Printf("skipping item %v: %v", item, err)
        continue
    }
}

// Pattern 3: accumulate errors
var errs []error
for _, item := range items {
    if err := process(item); err != nil {
        errs = append(errs, err)
    }
}
return errors.Join(errs...) // nil if errs is empty... wait, needs check
// Better:
if len(errs) > 0 { return errors.Join(errs...) }
return nil
```

## Panic and recover

Panic is for truly unrecoverable programmer errors — not for normal error flow.

```go
// panic with a value
panic("this should never happen")
panic(fmt.Sprintf("unexpected state: %v", state))

// recover — only works inside a deferred function
func safeRun(f func()) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic: %v\n%s", r, debug.Stack())
        }
    }()
    f()
    return nil
}

// Common use: convert panics at package boundaries
// Internal code panics, exported functions recover and return errors
```

## defer order

```go
func example() {
    defer fmt.Println("3")
    defer fmt.Println("2")
    defer fmt.Println("1")
    // prints: 1, 2, 3 — LIFO
}

// defer + named return = modify return value
func mustPositive(n int) (result int, err error) {
    defer func() {
        if err != nil {
            result = -1
        }
    }()
    if n < 0 {
        return 0, errors.New("negative")
    }
    return n, nil
}
```

## log/slog structured logging (Go 1.21+)

```go
import "log/slog"

// Default logger
slog.Info("user created", "id", 42, "name", "Alice")
slog.Error("request failed", "err", err, "path", r.URL.Path)

// Custom logger with JSON output
logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelDebug,
}))
logger.Info("server started", slog.Int("port", 8080))

// Add context to a logger
child := logger.With("requestID", reqID)
child.Debug("processing request")
```
