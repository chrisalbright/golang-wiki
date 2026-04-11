---
title: Context
type: concept
tags: [context, cancellation, deadlines, values]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

`context.Context` propagates cancellation, deadlines, and request-scoped values through call chains. Every blocking operation should accept a context. See also: [[Concurrency]], [[Error Handling]].

## Context basics

```go
import "context"

// Root contexts
ctx := context.Background() // top-level, never cancelled
ctx := context.TODO()       // placeholder during refactoring

// Derive contexts — parent cancels all children
ctx, cancel := context.WithCancel(parent)
defer cancel() // ALWAYS call cancel to avoid goroutine leaks

ctx, cancel := context.WithTimeout(parent, 5*time.Second)
defer cancel()

ctx, cancel := context.WithDeadline(parent, time.Now().Add(5*time.Second))
defer cancel()

// Go 1.21+: WithoutCancel — detach from parent cancellation
ctx = context.WithoutCancel(parent)

// Go 1.21+: AfterFunc — run function when context done
stop := context.AfterFunc(ctx, func() { cleanup() })
defer stop()
```

## Checking cancellation

```go
// ctx.Done() returns a channel closed when context is cancelled
select {
case <-ctx.Done():
    return ctx.Err() // context.Canceled or context.DeadlineExceeded
default:
    // continue working
}

// In a loop
func processItems(ctx context.Context, items []Item) error {
    for _, item := range items {
        if err := ctx.Err(); err != nil {
            return err // check before each iteration
        }
        if err := process(ctx, item); err != nil {
            return err
        }
    }
    return nil
}
```

## Passing context through the call chain

```go
// Context is always the first parameter by convention
func FetchUser(ctx context.Context, id int) (*User, error) {
    req, err := http.NewRequestWithContext(ctx, "GET",
        fmt.Sprintf("/users/%d", id), nil)
    if err != nil {
        return nil, err
    }
    resp, err := http.DefaultClient.Do(req)
    // ...
}

func GetUserOrders(ctx context.Context, userID int) ([]Order, error) {
    user, err := FetchUser(ctx, userID) // propagate same context
    if err != nil { return nil, err }
    return FetchOrders(ctx, user.ID)
}

// Database queries — context enables query cancellation
func QueryUsers(ctx context.Context, db *sql.DB) ([]User, error) {
    rows, err := db.QueryContext(ctx, "SELECT id, name FROM users")
    // ...
}
```

## Context values

Values should be request-scoped data: request IDs, auth tokens, trace spans.

```go
// Define typed key to avoid collisions
type contextKey string
const requestIDKey contextKey = "requestID"

// Store
ctx = context.WithValue(ctx, requestIDKey, "req-123")

// Retrieve
if id, ok := ctx.Value(requestIDKey).(string); ok {
    slog.Info("handling request", "requestID", id)
}

// Middleware pattern (HTTP)
func requestIDMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        id := r.Header.Get("X-Request-ID")
        if id == "" { id = uuid.New().String() }
        ctx := context.WithValue(r.Context(), requestIDKey, id)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

## Context with HTTP server (Go 1.22+)

```go
mux := http.NewServeMux()

// Go 1.22: pattern with method and path variables
mux.HandleFunc("GET /users/{id}", func(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    id := r.PathValue("id") // new in 1.22

    user, err := getUser(ctx, id)
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            http.Error(w, "not found", http.StatusNotFound)
            return
        }
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    json.NewEncoder(w).Encode(user)
})
```

## Rules for context

1. Pass context as the first argument, never store it in a struct
2. Never pass a nil context — use `context.Background()` or `context.TODO()`
3. Always call cancel when you're done — prevents goroutine leaks
4. Don't use context values for optional function parameters
5. Context values are for cross-cutting concerns, not business logic
