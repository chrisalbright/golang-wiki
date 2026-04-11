---
title: Key Third-Party Packages
type: entity
tags: [packages, dependencies, ecosystem]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Essential third-party packages for Go development, curated for quality and widespread adoption.

## HTTP / Web

| Package | Purpose |
|---------|---------|
| `github.com/gin-gonic/gin` | HTTP router and framework |
| `github.com/go-chi/chi/v5` | Lightweight router (idiomatic) |
| `github.com/labstack/echo/v4` | High-performance HTTP framework |

## Database

| Package | Purpose |
|---------|---------|
| `github.com/jmoiron/sqlx` | Extensions to database/sql |
| `github.com/jackc/pgx/v5` | PostgreSQL driver (preferred) |
| `entgo.io/ent` | Type-safe ORM / entity framework |
| `github.com/sqlc-dev/sqlc` | Generate type-safe Go from SQL |
| `github.com/pressly/goose/v3` | Database migrations |

## Testing

| Package | Purpose |
|---------|---------|
| `github.com/stretchr/testify` | Assertions and mocks |
| `github.com/vektra/mockery/v2` | Generate mocks from interfaces |
| `github.com/matryer/moq` | Minimal mock generation |

## Observability

| Package | Purpose |
|---------|---------|
| `go.opentelemetry.io/otel` | Distributed tracing (OTEL) |
| `github.com/prometheus/client_golang` | Prometheus metrics |

## Configuration

| Package | Purpose |
|---------|---------|
| `github.com/spf13/viper` | Config from files/env/flags |
| `github.com/knadh/koanf/v2` | Lightweight config |
| `github.com/caarlos0/env/v11` | Struct from env vars |

## CLI

| Package | Purpose |
|---------|---------|
| `github.com/spf13/cobra` | CLI framework |
| `github.com/urfave/cli/v3` | Alternative CLI framework |

## Concurrency

| Package | Purpose |
|---------|---------|
| `golang.org/x/sync` | errgroup, singleflight, semaphore |

## Encoding / Serialization

| Package | Purpose |
|---------|---------|
| `github.com/bytedance/sonic` | Fast JSON (drop-in for encoding/json) |
| `google.golang.org/protobuf` | Protocol buffers |
| `github.com/BurntSushi/toml` | TOML parsing |
| `gopkg.in/yaml.v3` | YAML parsing |

## Utilities

| Package | Purpose |
|---------|---------|
| `github.com/google/uuid` | UUID generation |
| `golang.org/x/exp` | Experimental (slices, maps, etc. — now in stdlib 1.21) |
| `github.com/samber/lo` | Lodash-style generic helpers |
