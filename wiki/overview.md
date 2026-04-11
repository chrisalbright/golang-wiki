---
title: Go Language — Overview
type: overview
tags: [go, overview]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Go (Golang) is a statically typed, compiled language designed at Google for simplicity, performance, and built-in concurrency. This wiki covers everything needed to become productive in modern Go (1.22+).

## Learning path

```
Basics → Types → Control flow → Functions
    ↓
Structs → Interfaces → Embedding
    ↓
Error handling → Panics → Defer
    ↓
Goroutines → Channels → Select → sync
    ↓
Generics → Context → io/fs
    ↓
Testing → Modules → Tooling → Idioms
```

## Core design philosophy

- **Simplicity over cleverness** — one obvious way to do things
- **Explicit over implicit** — no hidden control flow, no operator overloading
- **Composition over inheritance** — interfaces and embedding instead of class hierarchies
- **Errors as values** — errors are returned, not thrown
- **Built-in concurrency** — goroutines and channels are first-class

## Key wiki sections

| Section | Pages |
|---------|-------|
| [[Language Basics]] | variables, constants, control flow, functions |
| [[Types]] | primitives, slices, maps, pointers |
| [[Structs and Interfaces]] | structs, methods, interfaces, embedding |
| [[Error Handling]] | errors, sentinel values, wrapping, panic/recover |
| [[Concurrency]] | goroutines, channels, select, sync, errgroup |
| [[Generics]] | type parameters, constraints, type inference |
| [[Context]] | cancellation, deadlines, values |
| [[Testing]] | table tests, subtests, benchmarks, fuzzing |
| [[Modules]] | go.mod, workspaces, versioning |
| [[Standard Library]] | key packages overview |
| [[Go Idioms]] | patterns, anti-patterns, style |
| [[Tooling]] | go build, vet, staticcheck, govulncheck |

## Go version timeline (relevant)

| Version | Key addition |
|---------|-------------|
| 1.18 | Generics, fuzzing |
| 1.19 | atomic types, doc comments |
| 1.20 | comparable in generics, errors.Join |
| 1.21 | log/slog, slices/maps packages, min/max builtins |
| 1.22 | range over integers, loop variable fix, net/http.ServeMux patterns |
| 1.23 | range over iterators, unique package |
| 1.24 | weak pointers, sync.Map typed, finalizers |
