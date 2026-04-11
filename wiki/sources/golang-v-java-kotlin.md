---
title: "Source: Go vs Java/Kotlin for Experienced Engineers"
type: source
tags: [java, kotlin, comparison, learning-path, mindset]
created: 2026-04-11
updated: 2026-04-11
sources: [raw/golang-v-java-kotlin.md]
---

A principal-engineer-level guide to Go for developers with deep Java/Kotlin experience. Covers mindset shifts, gotchas, and a practical learning path.

## Key takeaways

1. **No classes/inheritance** — structs + methods + implicit interfaces replace the entire OOP hierarchy.

2. **Errors as values, not exceptions** — `if err != nil` everywhere. Verbose but explicit and auditable.

3. **Goroutines >> Java threads** — thousands of goroutines is normal; Java's `ExecutorService` is heavyweight by comparison.

4. **Single binary, no JVM** — `go build` → self-contained binary. No container overhead just to ship a runtime.

5. **`go mod` >> Maven/Gradle** — simple, fast, reproducible.

6. **Gotchas for Java devs**: forgetting `defer` for cleanup, nil receiver panics, value vs pointer semantics, slice aliasing, `%w` for error wrapping.

7. **Standard library is enough for most things** — `net/http`, `database/sql`, `encoding/json` handle the common cases.

## Recommended learning path (for experienced engineers)

1. A Tour of Go — https://go.dev/tour/ (1-2 days)
2. Effective Go — https://go.dev/doc/effective_go (1-2 days)
3. Go by Example — https://gobyexample.com/
4. *Learning Go* (Jon Bodner, O'Reilly) — best book for senior devs
5. *100 Go Mistakes and How to Avoid Them* — pitfall catalog
6. *Concurrency in Go* (Katherine Cox-Buday) — deep concurrency

## Linked pages

- [[Go for JVM Developers]] — full comparison and gotchas
- [[Concurrency]] — goroutines vs Java threads
- [[Modules]] — go mod vs Maven/Gradle
- [[Error Handling]] — errors as values vs exceptions
