---
title: "Source: Fakes in Golang"
type: source
tags: [testing, fakes, mocks, dependency-injection, interfaces]
created: 2026-04-11
updated: 2026-04-11
sources: [raw/fakes-in-golang.md]
---

Explains how Go handles test doubles (fakes/mocks) without a mocking framework. Targeted at engineers coming from Java/Kotlin with Mockito background.

## Key takeaways

1. **No dynamic mocking in Go** — no monkey-patching, no runtime mock generation without code gen tools. Go's design prevents it intentionally.

2. **Interface injection is the idiomatic solution** — wrap dependencies behind small interfaces, inject in constructors, swap in tests with hand-written fakes.

3. **Function type injection** — for single-method cases, a `func(int) int` field is lighter than defining an interface. Direct analog of Java's `IntUnaryOperator`.

4. **Seeded `*rand.Rand`** — when determinism matters without changing architecture, pass a `rand.New(rand.NewPCG(seed, seq))` instance.

5. **math/rand/v2 is preferred in Go 1.22+** — use `rand.IntN` not `rand.Intn`.

6. **Explicit dependency > hidden dependency** — Go's philosophy forces dependencies to be visible in struct fields and constructor signatures.

## Patterns ranked

| Pattern | When to use |
|---------|-------------|
| Interface injection | Multiple implementations, complex behavior |
| Function type injection | Single-method case, lightweight |
| Seeded rand | No design change needed, testing distribution |
| Global var swap | Avoid — fragile with `-race` |

## Linked pages

- [[Testing with Fakes]] — full pattern reference
- [[Testing]] — general testing guide
- [[Structs and Interfaces]] — interface design
