---
title: Concurrency
type: concept
tags: [concurrency, goroutines, channels, select, sync, errgroup]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Go's concurrency model: goroutines (lightweight threads) + channels (communication) + sync primitives. See also: [[Context]], [[Error Handling]].

## Goroutines

```go
// Launch a goroutine with go keyword
go func() {
    fmt.Println("running in background")
}()

// Goroutines are cheap — thousands are normal
// They start with ~8KB stack, grow as needed

// Wait for goroutines with WaitGroup
var wg sync.WaitGroup
for i := range 5 {
    wg.Add(1)
    go func(n int) {
        defer wg.Done()
        fmt.Println(n)
    }(i) // pass i as argument to avoid loop variable capture
}
wg.Wait()

// Go 1.22+: loop variable fix — each iteration has its own copy
// The explicit argument is no longer necessary but still idiomatic
for i := range 5 {
    go func() { fmt.Println(i) }() // safe in 1.22+
}
```

## Channels

```go
// Unbuffered channel — sender blocks until receiver is ready
ch := make(chan int)

go func() { ch <- 42 }() // send
v := <-ch                 // receive

// Buffered channel — sender blocks only when buffer full
ch := make(chan string, 10)
ch <- "hello" // doesn't block (buffer has space)

// Close a channel — signals no more values
close(ch)

// Receive from closed channel
v, ok := <-ch  // ok=false when channel closed and empty

// Range over channel until closed
for v := range ch {
    fmt.Println(v)
}

// Directional channel types (for function signatures)
func produce(out chan<- int) { out <- 1 } // send-only
func consume(in <-chan int)  { <-in }     // receive-only
```

## Select

Multiplexes over multiple channel operations.

```go
select {
case v := <-ch1:
    fmt.Println("ch1:", v)
case v := <-ch2:
    fmt.Println("ch2:", v)
case ch3 <- value:
    fmt.Println("sent to ch3")
case <-time.After(5 * time.Second):
    fmt.Println("timeout")
default:
    fmt.Println("no channels ready") // non-blocking
}

// Fan-out pattern
func fanOut(in <-chan int, n int) []<-chan int {
    outs := make([]<-chan int, n)
    for i := range n {
        out := make(chan int)
        outs[i] = out
        go func() {
            defer close(out)
            for v := range in { out <- v }
        }()
    }
    return outs
}

// Done channel pattern (superseded by context.Context)
done := make(chan struct{})
go func() {
    defer close(done)
    // work...
}()
<-done // wait for completion
```

## sync package

```go
// Mutex — mutual exclusion for shared state
type SafeCounter struct {
    mu sync.Mutex
    n  int
}
func (c *SafeCounter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.n++
}

// RWMutex — multiple readers, single writer
var mu sync.RWMutex
mu.RLock(); defer mu.RUnlock()  // read lock
mu.Lock();  defer mu.Unlock()   // write lock

// Once — run exactly once
var once sync.Once
once.Do(func() { /* initialization */ })

// sync.Map — concurrent map (use when reads >> writes or key sets disjoint)
var m sync.Map
m.Store("key", "value")
v, ok := m.Load("key")
m.LoadOrStore("key", "default")
m.Delete("key")
m.Range(func(k, v any) bool {
    fmt.Println(k, v)
    return true // continue
})

// atomic — lock-free operations on primitives
import "sync/atomic"
var counter atomic.Int64
counter.Add(1)
counter.Load()
counter.CompareAndSwap(old, new)
```

## errgroup — goroutines with error propagation

```go
import "golang.org/x/sync/errgroup"

// Basic usage
g, ctx := errgroup.WithContext(context.Background())

g.Go(func() error {
    return fetchUser(ctx)
})
g.Go(func() error {
    return fetchOrders(ctx)
})

if err := g.Wait(); err != nil {
    return err // first non-nil error from any goroutine
}

// SetLimit — bound parallelism
g.SetLimit(10) // max 10 goroutines at a time
for _, url := range urls {
    url := url
    g.Go(func() error { return fetch(ctx, url) })
}
```

## Common concurrency patterns

```go
// Worker pool
func workerPool(jobs <-chan Job, results chan<- Result, n int) {
    var wg sync.WaitGroup
    for range n {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                results <- process(job)
            }
        }()
    }
    wg.Wait()
    close(results)
}

// Pipeline
func pipeline(nums []int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums { out <- n }
    }()
    return out
}

// Timeout with context
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

select {
case result := <-compute(ctx):
    use(result)
case <-ctx.Done():
    return ctx.Err()
}
```

## Common mistakes

```go
// WRONG: goroutine leak — nothing to stop the goroutine
go func() {
    for { doWork() } // runs forever if nobody sends on done
}()

// RIGHT: use context for cancellation
go func(ctx context.Context) {
    for {
        select {
        case <-ctx.Done(): return
        default: doWork()
        }
    }
}(ctx)

// WRONG: data race — concurrent map writes
m := map[string]int{}
go func() { m["a"] = 1 }()
go func() { m["b"] = 2 }()

// RIGHT: sync.Map or Mutex
```
