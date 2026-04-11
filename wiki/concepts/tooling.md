---
title: Tooling
type: concept
tags: [tooling, build, vet, staticcheck, govulncheck, pprof]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Go's toolchain is built-in and comprehensive. See also: [[Modules]], [[Testing]].

## go build and run

```bash
# Build binary
go build ./...               # build all packages
go build -o myapp ./cmd/api  # named binary

# Run without building
go run ./cmd/api

# Cross-compile
GOOS=linux GOARCH=amd64 go build -o myapp-linux ./cmd/api
GOOS=darwin GOARCH=arm64 go build -o myapp-mac ./cmd/api
GOOS=windows GOARCH=amd64 go build -o myapp.exe ./cmd/api

# Build with version info via ldflags
go build -ldflags="-X main.version=1.2.3 -X main.commit=$(git rev-parse HEAD)" ./cmd/api

# Optimizations
go build -trimpath               # remove local paths from binary
go build -ldflags="-s -w"        # strip debug info (smaller binary)
```

## go vet — static analysis (built-in)

```bash
go vet ./...

# Common checks:
# - unreachable code
# - printf format mismatches
# - mutex copy
# - shadowed variables (with shadow analyzer)
# - incorrect struct tags
```

## Race detector

```bash
# Build or test with race detector
go test -race ./...
go run -race ./cmd/api

# MUST use during development and CI
# ~5-10x slowdown, finds data races at runtime
```

## staticcheck — extended linting

```bash
go install honnef.co/go/tools/cmd/staticcheck@latest
staticcheck ./...

# Checks:
# - deprecated API usage
# - unnecessary type conversions
# - unreachable code
# - incorrect string operations
# - performance issues
```

## golangci-lint — meta-linter

```bash
# Install
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
golangci-lint run

# .golangci.yml
linters:
  enable:
    - staticcheck
    - gosimple
    - errcheck
    - revive
    - govet
    - unused
```

## govulncheck — vulnerability scanner

```bash
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...

# Checks your code against Go vulnerability database
# Only reports vulns actually called by your code
```

## go generate

```bash
# Run code generators defined in source
//go:generate stringer -type=Status
//go:generate mockery --name=UserRepo

go generate ./...

# Common generators:
# stringer — String() for iota types
# mockery, moq — mock generation
# sqlc — type-safe SQL
# protoc — protobuf
```

## Profiling with pprof

```go
// Add to your HTTP server for live profiling
import _ "net/http/pprof"

go func() {
    log.Println(http.ListenAndServe("localhost:6060", nil))
}()
```

```bash
# CPU profile
go test -cpuprofile=cpu.prof -bench=.
go tool pprof cpu.prof

# Memory profile
go test -memprofile=mem.prof -bench=.
go tool pprof mem.prof

# Live from running server
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# In pprof interactive mode:
# top         - top functions by CPU
# list Func   - annotated source for function
# web         - open graph in browser
# pdf         - generate PDF report

# Trace — more detailed timeline
go test -trace=trace.out
go tool trace trace.out
```

## go doc and godoc

```bash
# View package docs in terminal
go doc fmt
go doc fmt.Sprintf
go doc os.File.Read

# Run local documentation server
go install golang.org/x/tools/cmd/godoc@latest
godoc -http=:6060
# Open http://localhost:6060
```

## Common project layout

```
myapp/
├── cmd/
│   └── api/
│       └── main.go         # entry point, thin wrapper
├── internal/               # private packages
│   ├── server/
│   ├── db/
│   └── auth/
├── pkg/                    # public packages (if any)
├── api/                    # OpenAPI specs, protobuf
├── testdata/               # test fixtures
├── Makefile
├── go.mod
├── go.sum
└── .golangci.yml
```

## Makefile patterns

```makefile
.PHONY: build test lint vet

build:
	go build -o bin/api ./cmd/api

test:
	go test -race ./...

bench:
	go test -bench=. -benchmem ./...

lint:
	golangci-lint run

vet:
	go vet ./...
	staticcheck ./...

vuln:
	govulncheck ./...

cover:
	go test -coverprofile=coverage.out ./...
	go tool cover -html=coverage.out
```
