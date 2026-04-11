---
title: Language Basics
type: concept
tags: [basics, variables, functions, control-flow]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Core syntax and constructs every Go program uses. See also: [[Types]], [[Functions]], [[Error Handling]].

## Variables

```go
// Short declaration (most common inside functions)
x := 42
name := "gopher"
ok := true

// Explicit type
var count int = 0
var message string

// Multiple assignment
a, b := 1, 2
a, b = b, a  // swap

// Blank identifier — discard unwanted values
_, err := strconv.Atoi("42")
```

## Constants

```go
const Pi = 3.14159
const MaxRetries = 3

// iota — auto-incrementing enumerator
type Direction int
const (
    North Direction = iota // 0
    East                   // 1
    South                  // 2
    West                   // 3
)

// Typed string constants for pseudo-enums
type Status string
const (
    StatusPending  Status = "pending"
    StatusComplete Status = "complete"
    StatusFailed   Status = "failed"
)
```

## Control flow

```go
// if — init statement scopes variable to the block
if err := doSomething(); err != nil {
    return err
}

// for — Go's only loop construct
for i := 0; i < 10; i++ { }   // C-style
for i < 10 { }                  // while-style
for { }                          // infinite loop

// range — iterate slices, maps, strings, channels
nums := []int{1, 2, 3}
for i, v := range nums { fmt.Println(i, v) }

// range over integer (Go 1.22+)
for i := range 5 { fmt.Println(i) } // 0,1,2,3,4

// switch — no fallthrough by default
switch status {
case "ok":
    handle()
case "retry", "timeout":
    retry()
default:
    fail()
}

// Type switch
switch v := any.(type) {
case int:
    fmt.Println("int:", v)
case string:
    fmt.Println("string:", v)
}
```

## Functions

```go
// Basic function
func add(a, b int) int {
    return a + b
}

// Multiple return values — idiomatic Go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Named return values (use sparingly)
func minMax(nums []int) (min, max int) {
    min, max = nums[0], nums[0]
    for _, n := range nums[1:] {
        if n < min { min = n }
        if n > max { max = n }
    }
    return // naked return
}

// Variadic functions
func sum(nums ...int) int {
    total := 0
    for _, n := range nums { total += n }
    return total
}
sum(1, 2, 3)
sum(nums...)  // unpack slice

// First-class functions
apply := func(f func(int) int, n int) int { return f(n) }
double := func(n int) int { return n * 2 }
apply(double, 5) // 10

// defer — runs when enclosing function returns (LIFO order)
func readFile(path string) error {
    f, err := os.Open(path)
    if err != nil { return err }
    defer f.Close()  // guaranteed to run
    // ... use f
    return nil
}
```

## init functions

```go
// Runs once at program startup, after package-level vars are initialized
func init() {
    // setup, validation, registration
}
```

## Zero values

Every type has a zero value — no uninitialized memory in Go:

| Type | Zero value |
|------|-----------|
| int, float64 | 0, 0.0 |
| bool | false |
| string | "" |
| pointer, slice, map, chan, func | nil |
| struct | all fields zero |
