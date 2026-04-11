---
title: Testing
type: concept
tags: [testing, table-tests, benchmarks, fuzzing, subtests, mocks]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Go has testing built into the toolchain — no external test framework needed. See also: [[Error Handling]], [[Modules]].

## Basic test

```go
// File: math_test.go  (must end in _test.go)
package math_test  // black-box test (external package)
// or: package math (white-box test, access unexported)

import (
    "testing"
    "yourmodule/math"
)

func TestAdd(t *testing.T) {
    got := math.Add(2, 3)
    want := 5
    if got != want {
        t.Errorf("Add(2,3) = %d, want %d", got, want)
    }
}
```

## Table-driven tests (idiomatic Go)

```go
func TestDivide(t *testing.T) {
    tests := []struct {
        name    string
        a, b    float64
        want    float64
        wantErr bool
    }{
        {"positive", 10, 2, 5.0, false},
        {"negative", -6, 3, -2.0, false},
        {"zero divisor", 5, 0, 0, true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Divide(tt.a, tt.b)
            if (err != nil) != tt.wantErr {
                t.Errorf("wantErr=%v, got err=%v", tt.wantErr, err)
            }
            if !tt.wantErr && got != tt.want {
                t.Errorf("got %v, want %v", got, tt.want)
            }
        })
    }
}

// Run: go test ./...
// Run specific: go test -run TestDivide/zero_divisor
```

## Assertions with testing/assert (stdlib alternative)

```go
// Standard library — manual checks only
// Popular third-party: testify (github.com/stretchr/testify)

import "github.com/stretchr/testify/assert"
import "github.com/stretchr/testify/require"

func TestSomething(t *testing.T) {
    result, err := doThing()
    require.NoError(t, err)          // stops test on failure
    assert.Equal(t, expected, result) // reports but continues
    assert.ErrorIs(t, err, ErrNotFound)
    assert.Contains(t, slice, item)
}
```

## Test helpers

```go
// t.Helper() marks function as helper — errors show caller's line
func assertUser(t *testing.T, got, want *User) {
    t.Helper()
    if got.ID != want.ID {
        t.Errorf("user ID: got %d, want %d", got.ID, want.ID)
    }
}

// t.Cleanup — register teardown
func TestWithDB(t *testing.T) {
    db := openTestDB(t)
    t.Cleanup(func() { db.Close() })
    // test...
}

// t.TempDir — cleaned up after test
dir := t.TempDir()

// t.Setenv — restored after test (Go 1.17+)
t.Setenv("MY_VAR", "test_value")
```

## Parallel tests

```go
func TestParallel(t *testing.T) {
    t.Parallel() // runs concurrently with other parallel tests

    tests := []struct{ name, input string }{ ... }
    for _, tt := range tests {
        tt := tt // Go <1.22: capture loop variable
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel()
            // ...
        })
    }
}
```

## Benchmarks

```go
func BenchmarkAdd(b *testing.B) {
    for range b.N { // b.N adjusted by framework
        Add(2, 3)
    }
}

// Setup before timing
func BenchmarkSort(b *testing.B) {
    data := generateData(1000)
    b.ResetTimer() // exclude setup
    for range b.N {
        b.StopTimer()
        s := slices.Clone(data) // fresh copy each time
        b.StartTimer()
        slices.Sort(s)
    }
}

// Run: go test -bench=. -benchmem
// Output: BenchmarkAdd-8  1000000000  0.29 ns/op
```

## Fuzz testing (Go 1.18+)

```go
func FuzzReverse(f *testing.F) {
    // Seed corpus
    f.Add("hello")
    f.Add("world")
    f.Add("")

    f.Fuzz(func(t *testing.T, s string) {
        rev := Reverse(s)
        rev2 := Reverse(rev)
        if s != rev2 {
            t.Errorf("double reverse != original: %q != %q", s, rev2)
        }
    })
}

// Run fuzz: go test -fuzz=FuzzReverse -fuzztime=30s
// Failures saved to testdata/fuzz/FuzzReverse/
```

## Mocking with interfaces

```go
// Define interface for the dependency
type UserRepo interface {
    GetUser(ctx context.Context, id int) (*User, error)
}

// Service under test
type UserService struct {
    repo UserRepo
}

// Mock implementation in tests
type mockUserRepo struct {
    users map[int]*User
}

func (m *mockUserRepo) GetUser(_ context.Context, id int) (*User, error) {
    u, ok := m.users[id]
    if !ok { return nil, ErrNotFound }
    return u, nil
}

func TestGetUser(t *testing.T) {
    repo := &mockUserRepo{users: map[int]*User{1: {ID: 1, Name: "Alice"}}}
    svc := &UserService{repo: repo}
    u, err := svc.GetUser(context.Background(), 1)
    // assert...
}

// Alternative: use mockery or moq for generated mocks
```

For deeper coverage of fake patterns — including function type injection and clock/rand faking — see [[Testing with Fakes]].

## Coverage

```go
// Run with coverage
go test -cover ./...
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out  // browser view
go tool cover -func=coverage.out  // per-function summary
```

## Test organization

```
package/
├── user.go
├── user_test.go         # white-box tests (same package)
├── user_integration_test.go
└── testdata/            # fixtures, golden files
```

```go
// Build tags to separate slow integration tests
//go:build integration

func TestDatabaseIntegration(t *testing.T) { ... }

// Run: go test -tags=integration ./...
```
