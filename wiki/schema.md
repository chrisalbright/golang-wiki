---
title: Wiki Schema & Conventions
type: concept
tags: [meta, conventions]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Conventions governing this Go learning wiki. All pages follow these rules.

## Page types

| Type | Location | Purpose |
|------|----------|---------|
| concept | concepts/ | Language features, patterns, idioms |
| entity | entities/ | Tools, packages, specifications |
| source | sources/ | One page per ingested raw source |
| query | queries/ | Filed synthesis from Q&A |
| overview | wiki/ | Big-picture synthesis |

## Frontmatter

```yaml
---
title: Page Title
type: concept | entity | source | query | overview
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [raw/filename.md]
---
```

## Cross-references

Use `[[PageTitle]]` wikilinks within prose. Every concept should link to related concepts.

## Code examples

- All examples use Go 1.22+ syntax unless noted otherwise
- Every code block has a language tag: ```go
- Prefer runnable, self-contained examples
- Annotate non-obvious lines with inline comments

## Go version policy

This wiki targets **Go 1.22+**. Features introduced in specific versions are noted inline, e.g. `(Go 1.21+)`.

## Tagging taxonomy

Core language tags: `basics`, `types`, `concurrency`, `errors`, `generics`, `testing`, `modules`, `stdlib`, `idioms`, `performance`, `tooling`
