# Go Learning Wiki — Index

Content catalog for the Go learning wiki. All pages with links and summaries.

## Overview

- [Go Language Overview](overview.md) — High-level synthesis, learning path, version timeline

## Concepts

### Fundamentals

- [Language Basics](concepts/language-basics.md) — Variables, constants, control flow, functions, defer, zero values
- [Types](concepts/types.md) — Primitives, slices, maps, pointers, type definitions, any
- [Structs and Interfaces](concepts/structs-and-interfaces.md) — Structs, methods, interfaces, embedding, functional options

### Error Handling & Safety

- [Error Handling](concepts/error-handling.md) — Errors as values, wrapping, sentinels, panic/recover, slog

### Concurrency

- [Concurrency](concepts/concurrency.md) — Goroutines, channels, select, sync, errgroup, patterns

### Modern Features

- [Generics](concepts/generics.md) — Type parameters, constraints, type inference, stdlib generics (Go 1.18+)
- [Context](concepts/context.md) — Cancellation, deadlines, values, HTTP integration

### Development

- [Testing](concepts/testing.md) — Table tests, subtests, benchmarks, fuzzing, mocks, coverage
- [Testing with Fakes](concepts/testing-with-fakes.md) — Interface injection, function type injection, clock/rand faking (2 sources)
- [Modules](concepts/modules.md) — go.mod, dependency management, workspaces, toolchains
- [Tooling](concepts/tooling.md) — Build, vet, race detector, staticcheck, govulncheck, pprof
- [Standard Library](concepts/standard-library.md) — net/http, encoding/json, strings, os, time, slog
- [Go Idioms](concepts/go-idioms.md) — Naming, patterns, anti-patterns, effective Go principles

### For JVM Developers

- [Go for JVM Developers](concepts/go-for-jvm-devs.md) — Mindset shifts, Java/Kotlin comparisons, gotchas, learning path (1 source)

## Entities

- [Go Language Specification](entities/go-spec.md) — Spec highlights: assignability, comparability, zero values
- [Key Third-Party Packages](entities/key-packages.md) — Curated ecosystem: HTTP, DB, testing, observability, CLI

## Sources

- [Fakes in Golang](sources/fakes-in-golang.md) — DI patterns for test doubles in Go
- [Go vs Java/Kotlin](sources/golang-v-java-kotlin.md) — Guide for experienced JVM engineers

## Wiki Meta

- [Schema & Conventions](schema.md) — Page types, frontmatter, code style, tagging
- [Log](log.md) — Operation history

---
*Last updated: 2026-04-11 | 14 concept pages | 2 entity pages | 2 source pages*
