---
title: Standard Library
type: concept
tags: [stdlib, http, json, io, strings, time, os]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Key standard library packages every Go developer uses. See also: [[Error Handling]], [[Concurrency]], [[Context]].

## net/http — HTTP server and client

```go
// Server (Go 1.22+ patterns with method routing)
mux := http.NewServeMux()

mux.HandleFunc("GET /health", func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    fmt.Fprint(w, "ok")
})

mux.HandleFunc("POST /users", createUser)
mux.HandleFunc("GET /users/{id}", getUser)  // path variable

// Path variable extraction
func getUser(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id") // Go 1.22+
    // ...
}

// Start server
srv := &http.Server{
    Addr:         ":8080",
    Handler:      mux,
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  60 * time.Second,
}
if err := srv.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
    log.Fatal(err)
}

// Graceful shutdown
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt)
defer stop()
<-ctx.Done()

shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()
srv.Shutdown(shutdownCtx)
```

```go
// HTTP Client
client := &http.Client{Timeout: 10 * time.Second}

req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
req.Header.Set("Authorization", "Bearer "+token)

resp, err := client.Do(req)
if err != nil { return err }
defer resp.Body.Close()

if resp.StatusCode != http.StatusOK {
    return fmt.Errorf("unexpected status: %d", resp.StatusCode)
}

body, err := io.ReadAll(resp.Body)
```

## encoding/json

```go
// Marshal struct to JSON
type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
}

data, err := json.Marshal(User{ID: 1, Name: "Alice"})
// {"id":1,"name":"Alice"}

// Unmarshal JSON to struct
var u User
err = json.Unmarshal(data, &u)

// Stream encode/decode (efficient for large data)
json.NewEncoder(w).Encode(u)
json.NewDecoder(r.Body).Decode(&u)

// Dynamic JSON with map
var m map[string]any
json.Unmarshal(data, &m)

// Pretty print
data, _ = json.MarshalIndent(u, "", "  ")
```

## strings and strconv

```go
import "strings"

strings.Contains("gopher", "go")     // true
strings.HasPrefix("gopher", "go")    // true
strings.HasSuffix("gopher", "er")    // true
strings.Count("gopher", "o")         // 1
strings.Replace("foo bar", "o", "0", -1) // "f00 bar" (-1=all)
strings.ToUpper("hello")             // "HELLO"
strings.TrimSpace("  hello  ")       // "hello"
strings.Split("a,b,c", ",")         // ["a","b","c"]
strings.Join([]string{"a","b"}, ",") // "a,b"
strings.Fields("  foo bar  ")        // ["foo","bar"]

// Builder — efficient string construction
var b strings.Builder
for _, word := range words {
    fmt.Fprintf(&b, "%s ", word)
}
result := b.String()

import "strconv"
n, err := strconv.Atoi("42")        // string → int
s := strconv.Itoa(42)               // int → string
f, err := strconv.ParseFloat("3.14", 64)
b, err := strconv.ParseBool("true")
s = strconv.FormatFloat(3.14, 'f', 2, 64) // "3.14"
```

## os and io

```go
import "os"

// Read file
data, err := os.ReadFile("path/to/file")

// Write file
err = os.WriteFile("out.txt", data, 0o644)

// Open for streaming
f, err := os.Open("file.txt")
defer f.Close()
scanner := bufio.NewScanner(f)
for scanner.Scan() {
    line := scanner.Text()
}

// Environment
val := os.Getenv("MY_VAR")
os.Setenv("MY_VAR", "value")

// Temp file
f, err := os.CreateTemp("", "prefix-*.txt")
defer os.Remove(f.Name())

// io/fs — virtual filesystem (Go 1.16+)
import "io/fs"
err = fs.WalkDir(os.DirFS("."), ".", func(path string, d fs.DirEntry, err error) error {
    if err != nil { return err }
    if !d.IsDir() { fmt.Println(path) }
    return nil
})
```

## time

```go
now := time.Now()
t := time.Date(2026, time.January, 1, 0, 0, 0, 0, time.UTC)

// Format — Go uses reference time: Mon Jan 2 15:04:05 MST 2006
t.Format("2006-01-02")           // "2026-01-01"
t.Format(time.RFC3339)           // "2026-01-01T00:00:00Z"
t.Format("15:04:05")             // "00:00:00"

// Parse
t, err := time.Parse("2006-01-02", "2026-01-01")
t, err = time.Parse(time.RFC3339, "2026-01-01T00:00:00Z")

// Durations
d := 5 * time.Second
d += 100 * time.Millisecond
time.Sleep(d)

// Timers and tickers
timer := time.NewTimer(5 * time.Second)
defer timer.Stop()
<-timer.C // block until fires

ticker := time.NewTicker(1 * time.Second)
defer ticker.Stop()
for range ticker.C { doWork() }

// Measure elapsed time
start := time.Now()
elapsed := time.Since(start)
```

## sync/atomic and math/rand

```go
// Seeded random (Go 1.20+: global source is auto-seeded)
n := rand.Int()
n = rand.Intn(100)          // [0, 100)
f := rand.Float64()         // [0.0, 1.0)
rand.Shuffle(len(s), func(i, j int) { s[i], s[j] = s[j], s[i] })

// Reproducible (for tests)
r := rand.New(rand.NewSource(42))
r.Intn(100)
```

## log/slog (Go 1.21+)

```go
// Default (text) logger
slog.Info("started", "port", 8080)
slog.Debug("detail", "key", value)
slog.Warn("slow", "latency", d)
slog.Error("failed", "err", err)

// JSON logger
logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
slog.SetDefault(logger)

// Structured attributes
slog.Info("request",
    slog.String("method", r.Method),
    slog.Int("status", 200),
    slog.Duration("latency", elapsed),
)

// Logger with default fields
logger = logger.With("service", "api", "version", "1.0")
```
