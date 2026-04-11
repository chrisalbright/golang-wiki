---
title: Go Idioms
type: concept
tags: [idioms, patterns, style, anti-patterns]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Patterns and style conventions idiomatic to Go. See also: [[Structs and Interfaces]], [[Error Handling]], [[Concurrency]].

## Naming conventions

```go
// Short, clear names — context disambiguates
// Not: userAccountRepository → repo
// Not: getUserByIdFromDatabase → getUser

// Acronyms stay uppercase
type JSONParser struct{}
type HTTPClient struct{}
func parseURL(s string) {}

// Interface names: verb + er (usually)
type Reader interface { Read([]byte) (int, error) }
type Formatter interface { Format() string }
type Validator interface { Validate() error }

// Unexported for package-private, exported for public API
type internalState struct{} // unexported
type PublicConfig struct{}  // exported

// Test files can access unexported via same package
// External tests use _test suffix on package name
```

## The comma-ok idiom

```go
// Map lookup
v, ok := m["key"]

// Type assertion
v, ok := iface.(ConcreteType)

// Channel receive
v, ok := <-ch // ok=false when closed

// Interface nil check
var i interface{} = (*MyType)(nil)
_, isNil := i.(*MyType) // true but i != nil — trap!
```

## Table-driven design

```go
// Register behavior in a map, not a switch
var handlers = map[string]func(w http.ResponseWriter, r *http.Request){
    "create": handleCreate,
    "update": handleUpdate,
    "delete": handleDelete,
}

// Register processors
var processors = map[string]Processor{
    "json": &JSONProcessor{},
    "xml":  &XMLProcessor{},
    "csv":  &CSVProcessor{},
}
```

## Error wrapping idiom

```go
// Always add context when returning errors
func processOrder(id string) error {
    order, err := db.GetOrder(id)
    if err != nil {
        return fmt.Errorf("processOrder %s: %w", id, err)
    }
    // ...
}

// Error chain reads naturally: "processOrder 42: db.GetOrder: connection refused"
```

## Constructor pattern

```go
// Use New* functions — not exported struct literals
type Config struct {
    timeout time.Duration
    retries int
}

func NewConfig(opts ...Option) *Config {
    c := &Config{
        timeout: 30 * time.Second,
        retries: 3,
    }
    for _, o := range opts { o(c) }
    return c
}

// Validate in constructor, not at every use site
func NewServer(cfg *Config) (*Server, error) {
    if cfg == nil {
        return nil, errors.New("nil config")
    }
    // ...
}
```

## Closures for state

```go
// Closure captures variables — use for handlers, callbacks, iterators
func makeAdder(x int) func(int) int {
    return func(y int) int { return x + y }
}

add5 := makeAdder(5)
add5(3) // 8

// Middleware factory pattern
func withLogging(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)
        slog.Info("request", "path", r.URL.Path, "duration", time.Since(start))
    })
}
```

## Sentinel pattern for configuration

```go
// Use zero value as "not set" or use explicit Option types
type Config struct {
    MaxRetries int           // 0 = use default
    Timeout    time.Duration // 0 = use default
    Logger     *slog.Logger  // nil = use default
}

func (c *Config) retries() int {
    if c.MaxRetries > 0 { return c.MaxRetries }
    return 3 // default
}
```

## Avoid these anti-patterns

```go
// ANTI: panic for expected errors
func getUser(id int) *User {
    u, err := db.Get(id)
    if err != nil { panic(err) } // don't do this
    return u
}

// ANTI: returning error AND logging it (double reporting)
if err := doThing(); err != nil {
    log.Printf("doThing: %v", err) // log here...
    return err                      // ...and propagate — caller will log too
}
// DO: either log OR return, not both

// ANTI: empty interface everywhere
func process(v interface{}) interface{} { ... } // loses type safety

// ANTI: ignoring errors
doThing() // if doThing returns error, check it

// ANTI: init() for complex logic
func init() { setupDatabase() } // hard to test, hidden coupling

// ANTI: global mutable state
var globalDB *sql.DB // makes testing hard; pass db explicitly instead
```

## io.Reader/Writer composition

```go
// Compose readers/writers for streaming
func compressAndEncrypt(r io.Reader, w io.Writer, key []byte) error {
    enc, err := cipher.NewWriter(w, key)
    if err != nil { return err }
    defer enc.Close()

    gz := gzip.NewWriter(enc)
    defer gz.Close()

    _, err = io.Copy(gz, r)
    return err
}

// Wrap for transformation
r := io.LimitReader(resp.Body, 1<<20)     // max 1MB
r = io.MultiReader(header, r)              // prepend data
w := io.MultiWriter(os.Stdout, logFile)    // tee output
```

## Iterators (Go 1.23+)

```go
import "iter"

// Range function — yield pattern
func Fibonacci() iter.Seq[int] {
    return func(yield func(int) bool) {
        a, b := 0, 1
        for {
            if !yield(a) { return }
            a, b = b, a+b
        }
    }
}

for n := range Fibonacci() {
    if n > 100 { break }
    fmt.Println(n)
}

// Two-value iterator
func Enumerate[T any](s []T) iter.Seq2[int, T] {
    return func(yield func(int, T) bool) {
        for i, v := range s {
            if !yield(i, v) { return }
        }
    }
}
```

## Effective Go principles

- **Don't communicate by sharing memory; share memory by communicating** (channels)
- **Accept interfaces, return concrete types**
- **Make the zero value useful** (sync.Mutex, bytes.Buffer work without initialization)
- **Errors are values** — handle them, don't ignore them
- **Clear is better than clever** — write for the reader
