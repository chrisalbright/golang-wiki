---
title: Generics
type: concept
tags: [generics, type-parameters, constraints, type-inference]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Generics (Go 1.18+) add type parameters to functions and types, enabling type-safe reuse without `any` and type assertions. See also: [[Types]], [[Structs and Interfaces]].

## Type parameters

```go
// Generic function — works on any ordered type
func Min[T constraints.Ordered](a, b T) T {
    if a < b { return a }
    return b
}

Min(3, 5)           // int inferred
Min(3.14, 2.71)     // float64 inferred
Min[string]("a", "b") // explicit

// Generic type
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(v T) { s.items = append(s.items, v) }
func (s *Stack[T]) Pop() (T, bool) {
    var zero T
    if len(s.items) == 0 { return zero, false }
    v := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return v, true
}

s := Stack[int]{}
s.Push(1); s.Push(2)
v, _ := s.Pop() // 2
```

## Constraints

```go
import "golang.org/x/exp/constraints"

// Built-in constraints (golang.org/x/exp/constraints or cmp package)
constraints.Ordered   // all types that support < > <= >=
constraints.Integer   // all integer types
constraints.Float     // all float types

// Custom constraint — union of types
type Number interface {
    int | int8 | int16 | int32 | int64 |
    float32 | float64
}

func Sum[T Number](nums []T) T {
    var total T
    for _, n := range nums { total += n }
    return total
}

// Tilde ~ — includes all types with underlying type
type Addable interface {
    ~int | ~float64  // includes type Celsius float64, etc.
}

// Structural constraint — require methods
type Stringer interface {
    String() string
}

func Print[T Stringer](v T) { fmt.Println(v.String()) }

// Combine type set + methods
type StringerInt interface {
    ~int
    String() string
}
```

## Type inference

```go
// Go infers type parameters from arguments
func Map[T, U any](s []T, f func(T) U) []U {
    result := make([]U, len(s))
    for i, v := range s { result[i] = f(v) }
    return result
}

nums := []int{1, 2, 3}
strs := Map(nums, strconv.Itoa) // T=int, U=string inferred

// Inference doesn't always work — be explicit when needed
// Especially for return-type-only type parameters
```

## Standard library generic functions (Go 1.21+)

```go
import (
    "slices"
    "maps"
    "cmp"
)

// slices package
slices.Contains(s, "go")
slices.Index(s, "go")             // -1 if not found
slices.Sort(s)                     // in-place sort
slices.SortFunc(s, cmp.Compare)
slices.Reverse(s)
slices.Max(s); slices.Min(s)
slices.Equal(a, b)
slices.Compact(s)                  // remove adjacent duplicates
slices.Collect(iter)               // iterator to slice (Go 1.23+)

// maps package
maps.Keys(m)                       // iterator over keys
maps.Values(m)
maps.Clone(m)
maps.Equal(m1, m2)
maps.DeleteFunc(m, func(k, v) bool { return v == 0 })

// cmp package
cmp.Compare(a, b)                  // -1, 0, 1
cmp.Or(a, b, c)                    // first non-zero value
```

## Practical generic patterns

```go
// Result type — generic either
type Result[T any] struct {
    Value T
    Err   error
}

func (r Result[T]) Unwrap() T {
    if r.Err != nil { panic(r.Err) }
    return r.Value
}

// Optional / nullable
type Option[T any] struct {
    value T
    some  bool
}

func Some[T any](v T) Option[T] { return Option[T]{value: v, some: true} }
func None[T any]() Option[T]    { return Option[T]{} }

func (o Option[T]) Unwrap() (T, bool) { return o.value, o.some }

// Filter, Map, Reduce
func Filter[T any](s []T, keep func(T) bool) []T {
    var out []T
    for _, v := range s {
        if keep(v) { out = append(out, v) }
    }
    return out
}

func Reduce[T, U any](s []T, init U, f func(U, T) U) U {
    acc := init
    for _, v := range s { acc = f(acc, v) }
    return acc
}

// Set type
type Set[T comparable] map[T]struct{}

func (s Set[T]) Add(v T)            { s[v] = struct{}{} }
func (s Set[T]) Contains(v T) bool  { _, ok := s[v]; return ok }
func (s Set[T]) Delete(v T)         { delete(s, v) }
```

## When NOT to use generics

```go
// Don't over-generify: if you only have one type, skip generics
// Don't use generics for simple runtime polymorphism — use interfaces

// Generics shine for:
// - Containers (Stack, Queue, Set, Result, Option)
// - Algorithms that work on any slice/map (sort, filter, map, reduce)
// - Type-safe adapters and wrappers
// - Functions that operate on comparable or ordered types

// Interfaces shine for:
// - Behavior abstraction (multiple types do the same thing differently)
// - Plugin-style extensibility
// - io.Reader/Writer patterns
```
