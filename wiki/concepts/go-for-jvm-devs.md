---
title: Go for JVM Developers
type: concept
tags: [java, kotlin, comparison, mindset, gotchas, learning-path]
created: 2026-04-11
updated: 2026-04-11
sources: [raw/golang-v-java-kotlin.md]
---

A translation guide for engineers with deep Java/Kotlin experience. Go feels like "C with modern conveniences" — less ceremony, faster compiles, no JVM tuning. See also: [[Go Idioms]], [[Concurrency]], [[Error Handling]], [[Testing with Fakes]].

## Core mindset shifts

| Java/Kotlin concept | Go equivalent |
|---------------------|---------------|
| Class with methods | Struct + methods on struct |
| Interface (explicit) | Interface (implicit / duck typing) |
| Abstract class / inheritance | Embedding + interface composition |
| `extends` / `implements` | Just implement the methods |
| Checked exceptions | Return `(result, error)` |
| `try/catch/finally` | `if err != nil` + `defer` |
| `synchronized` / `ReentrantLock` | `sync.Mutex` / `sync.RWMutex` |
| `ExecutorService` / thread pool | Goroutines + `errgroup` |
| `CompletableFuture` / coroutine | Channel + goroutine |
| Generics `<T extends Comparable>` | `[T constraints.Ordered]` |
| Annotation processing | `go generate` + code gen tools |
| Spring Boot DI | Constructor injection via interfaces |
| Maven/Gradle | `go mod` (much simpler) |
| JAR/WAR | Single static binary |

## Concurrency: goroutines vs threads

```go
// Java: ExecutorService + Future — heavyweight, pool-managed
// ExecutorService pool = Executors.newFixedThreadPool(10);
// Future<Result> f = pool.submit(() -> expensiveWork());

// Go: goroutine — 8KB initial stack, thousands are normal
go expensiveWork()   // fire and forget

// With result collection (errgroup = structured concurrency)
g, ctx := errgroup.WithContext(context.Background())
results := make([]Result, len(items))
for i, item := range items {
    i, item := i, item
    g.Go(func() error {
        r, err := process(ctx, item)
        results[i] = r
        return err
    })
}
if err := g.Wait(); err != nil { return err }
```

## Error handling vs exceptions

```go
// Java: throw new UserNotFoundException(id)
// Kotlin: throw UserNotFoundException(id)

// Go: return error value
func getUser(id int) (*User, error) {
    u, err := db.Query(id)
    if err != nil {
        return nil, fmt.Errorf("getUser %d: %w", id, err)
    }
    return u, nil
}

// Caller must handle — no silent propagation
u, err := getUser(42)
if err != nil {
    // handle explicitly — cannot be ignored by accident
}

// Check error type (like instanceof in Java)
var nfe *NotFoundError
if errors.As(err, &nfe) {
    // respond with 404
}
```

## Interfaces: implicit vs explicit

```go
// Java: class Dog implements Animal { ... }
// Go: just implement the methods — no declaration needed

type Animal interface {
    Speak() string
}

type Dog struct{ Name string }
func (d Dog) Speak() string { return "Woof" } // Dog satisfies Animal implicitly

// This means any type can satisfy any interface retroactively
// — great for adapting third-party types
```

## No inheritance — use embedding

```go
// Java:
// class AdminUser extends User { ... }

// Go: embed the type
type User struct {
    ID   int
    Name string
}
func (u User) Greet() string { return "Hello, " + u.Name }

type AdminUser struct {
    User             // embedded — promotes fields and methods
    Permissions []string
}

a := AdminUser{User: User{1, "Alice"}, Permissions: []string{"read", "write"}}
a.Greet() // promoted from User
a.Name    // promoted field
```

## Dependency injection: no Spring needed

```go
// Java: @Autowired / @Bean / Spring container
// Go: pass dependencies explicitly in constructors

type Server struct {
    db     *sql.DB
    logger *slog.Logger
    cache  Cache  // interface
}

func NewServer(db *sql.DB, logger *slog.Logger, cache Cache) *Server {
    return &Server{db: db, logger: logger, cache: cache}
}

// Wire everything in main()
func main() {
    db, _ := sql.Open("postgres", os.Getenv("DATABASE_URL"))
    logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
    cache := redis.NewCache(os.Getenv("REDIS_URL"))
    srv := NewServer(db, logger, cache)
    srv.Run()
}
```

## Common gotchas for Java developers

```go
// 1. Forgetting defer for cleanup (no try-with-resources)
f, err := os.Open("file.txt")
if err != nil { return err }
defer f.Close()  // MUST add this — no AutoCloseable

// 2. Nil receiver — method on nil pointer doesn't always panic
var u *User  // nil pointer
u.Name       // PANIC: nil pointer dereference
// But methods CAN work on nil receiver if designed for it:
func (u *User) IsNil() bool { return u == nil } // safe

// 3. Slice aliasing — slices share underlying array
a := []int{1, 2, 3}
b := a[1:]      // b shares a's array
b[0] = 99       // modifies a[1] too!
b = slices.Clone(b) // break the aliasing

// 4. Map read is safe on nil, write panics
var m map[string]int
_ = m["key"]   // ok: returns 0
m["key"] = 1   // PANIC: assignment to nil map

// 5. Error wrapping with %w (not %v)
err = fmt.Errorf("context: %w", err) // %w preserves errors.Is/As chain
err = fmt.Errorf("context: %v", err) // %v loses the chain

// 6. Loop variable capture (Go <1.22)
for _, v := range items {
    go func() { use(v) }() // captures reference, not value in Go <1.22
    // Fix: go func(v Item) { use(v) }(v)
    // Go 1.22+: fixed automatically
}
```

## Performance expectations

- Compile: seconds for large projects (vs minutes for Gradle)
- Binary: 5-15MB static binary, no runtime required
- Memory: lower baseline than JVM (no GC warmup, no class loading)
- Throughput: matches or beats Java for most server workloads
- GC latency: sub-millisecond pauses (Go GC is non-generational, tuned for low latency)

## Learning path for experienced engineers

1. **A Tour of Go** — https://go.dev/tour/ (1-2 days, do all exercises)
2. **Effective Go** — https://go.dev/doc/effective_go (1-2 days)
3. **Go by Example** — https://gobyexample.com/ (reference as you build)
4. **Learn Go with Tests** — https://quii.gitbook.io/learn-go-with-tests/ (TDD approach)
5. **Book:** *Learning Go* (Jon Bodner, O'Reilly) — best for senior engineers
6. **Book:** *100 Go Mistakes* — pitfall catalog for experienced devs
7. **Book:** *Concurrency in Go* (Cox-Buday) — once comfortable with basics
8. Visual roadmap: https://roadmap.sh/golang

## First projects to build

- CLI tool with `flag` or `cobra`
- HTTP API with `net/http` + `encoding/json` (no framework)
- Concurrent worker pool — feel the goroutine difference
- Port a small Java service — notice what disappears (boilerplate) and what appears (explicit error handling)
