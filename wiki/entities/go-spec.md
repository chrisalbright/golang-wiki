---
title: Go Language Specification
type: entity
tags: [spec, reference, go]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

The official Go language specification defines the language semantics. Current version targets Go 1.22+.

## Key specification sections

- **Types** — type identity, assignability, comparability rules
- **Expressions** — operator precedence, evaluation order
- **Statements** — defer semantics, select fairness
- **Built-in functions** — make, new, append, copy, delete, len, cap, close
- **Packages** — init order, blank identifier

## Important specification details

### Assignability

A value is assignable to a variable of type T if:
- Types are identical
- Types have identical underlying types AND at least one is not a named type
- T is an interface that V implements
- T is a channel, V is a bidirectional channel, and identical element types

### Comparability

- **Comparable**: bool, int, float, complex, string, pointer, channel, interface, arrays of comparable
- **Not comparable** (use reflect.DeepEqual for tests): slice, map, func

```go
// Comparable — can use as map key or in ==
type Point struct{ X, Y int }
m := map[Point]string{}

// Not comparable — compile error as map key
type Slice struct{ data []int }
// map[Slice]string{} → compile error
```

### Zero values matter

Every type has a meaningful zero value — no uninitialized memory trap:
```go
var mu sync.Mutex      // ready to use
var b bytes.Buffer     // ready to use
var wg sync.WaitGroup  // ready to use
```

### defer execution order

Deferred functions run after return value is set but before function returns. Named return values CAN be modified by deferred functions.

## Reference links

- Go spec: https://go.dev/ref/spec
- Effective Go: https://go.dev/doc/effective_go
- Go blog: https://go.dev/blog/
- Go playground: https://play.golang.org
