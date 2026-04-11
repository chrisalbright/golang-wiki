---
title: Modules
type: concept
tags: [modules, go.mod, dependencies, versioning, workspaces]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Go modules are the standard dependency management system (since Go 1.11, mandatory since 1.16). See also: [[Tooling]], [[Testing]].

## go.mod

```
module github.com/user/myapp

go 1.22

require (
    github.com/gin-gonic/gin v1.9.1
    golang.org/x/sync v0.7.0
)

require (
    // indirect dependencies
    github.com/bytedance/sonic v1.11.6 // indirect
)
```

## Module commands

```bash
# Initialize a new module
go mod init github.com/user/myapp

# Add missing, remove unused dependencies
go mod tidy

# Download dependencies to local cache
go mod download

# Show dependency graph
go mod graph

# Verify downloaded packages match checksums
go mod verify

# Upgrade all dependencies to latest patch
go get -u=patch ./...

# Upgrade specific dependency
go get github.com/gin-gonic/gin@v1.10.0
go get github.com/gin-gonic/gin@latest

# Pin to a specific commit
go get github.com/user/repo@abc1234

# Remove a dependency
go get github.com/pkg/errors@none
```

## go.sum

```
# Checksums for every dependency version — commit this file
# Do NOT edit manually
github.com/gin-gonic/gin v1.9.1 h1:4idEAncQnU5cB7BeOkPtxjfCSye0AAm1R0RVIqJ+Jmg=
github.com/gin-gonic/gin v1.9.1/go.mod h1:hPrL7YrpYKXt5YId3A/Tnip5kqbEAP+KLuI3SUcPTeU=
```

## Versioning

Go modules use semantic versioning. v2+ modules require a path suffix.

```
// v1 import (normal)
import "github.com/user/repo"

// v2+ import (major version in path)
import "github.com/user/repo/v2"

// go.mod for v2 module
module github.com/user/repo/v2
```

## Workspaces (Go 1.18+)

Workspaces let you work on multiple modules simultaneously without replace directives.

```bash
# Create workspace
go work init ./myapp ./mylib

# Add module to workspace
go work use ./newmodule

# Sync workspace
go work sync
```

```
# go.work file
go 1.22

use (
    ./myapp
    ./mylib
    ./newmodule
)
```

## Replace directives

```
# go.mod — local development override
replace github.com/user/lib => ../lib

# Fork redirect
replace github.com/original/pkg => github.com/fork/pkg v1.2.3

# Remove replace directives before releasing!
```

## Module proxy and GOPATH

```bash
# GOPROXY — where to fetch modules (default: proxy.golang.org)
export GOPROXY=https://proxy.golang.org,direct

# GONOSUMCHECK / GONOSUMDB — skip sum check for private modules
export GONOSUMDB=gitlab.mycompany.com/*
export GOPRIVATE=gitlab.mycompany.com/*  # sets GONOSUMDB + GONOPROXY

# Vendor — copy dependencies into vendor/ for offline builds
go mod vendor
go build -mod=vendor ./...
```

## Internal packages

```
myapp/
├── internal/         # only importable within myapp module
│   └── db/
│       └── db.go
└── api/
    └── handler.go    # can import internal/db, outside cannot
```

## Toolchain management (Go 1.21+)

```
# go.mod — pin exact Go version
module myapp
go 1.22.3
toolchain go1.22.3

# Download and use a specific toolchain
go get toolchain@go1.22.3
```
