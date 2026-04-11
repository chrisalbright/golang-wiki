---
title: Structs and Interfaces
type: concept
tags: [structs, interfaces, embedding, methods, composition]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Go uses structs and interfaces for composition — no classes or inheritance. See also: [[Types]], [[Error Handling]], [[Generics]].

## Structs

```go
// Definition
type User struct {
    ID        int
    Name      string
    Email     string
    CreatedAt time.Time
}

// Literal — always name fields
u := User{
    ID:    1,
    Name:  "Alice",
    Email: "alice@example.com",
    CreatedAt: time.Now(),
}

// Pointer to struct (common for mutation)
u := &User{Name: "Bob"}
u.Name = "Robert"  // auto-dereferenced, no need for (*u).Name

// Anonymous struct (one-off data shapes, JSON tests)
point := struct{ X, Y int }{X: 10, Y: 20}
```

## Methods

```go
// Value receiver — for read-only, small structs
func (u User) String() string {
    return fmt.Sprintf("%s <%s>", u.Name, u.Email)
}

// Pointer receiver — for mutation or large structs
func (u *User) SetEmail(email string) {
    u.Email = email
}

// Convention: pick one receiver type per struct and stick with it
// If any method needs pointer receiver, make them all pointer receivers
```

## Struct tags

```go
type Product struct {
    ID    int     `json:"id"              db:"product_id"`
    Name  string  `json:"name"            db:"name"`
    Price float64 `json:"price,omitempty" db:"price"`
    // omitempty — omit field from JSON if zero value
}

// Read tags at runtime via reflect
t := reflect.TypeOf(Product{})
field, _ := t.FieldByName("Name")
fmt.Println(field.Tag.Get("json")) // "name"
```

## Embedding

Embedding composes types — promoted fields and methods appear directly on the outer struct.

```go
type Animal struct {
    Name string
}

func (a Animal) Speak() string { return a.Name + " speaks" }

type Dog struct {
    Animal        // embedded — NOT a field name
    Breed  string
}

d := Dog{Animal: Animal{Name: "Rex"}, Breed: "Lab"}
fmt.Println(d.Name)    // promoted field
fmt.Println(d.Speak()) // promoted method

// Outer type can override promoted methods
func (d Dog) Speak() string { return d.Name + " barks" }
```

## Interfaces

Interfaces are satisfied implicitly — no `implements` keyword.

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}

type Stringer interface {
    String() string
}

// Compose interfaces
type ReadWriter interface {
    io.Reader
    io.Writer
}

// Any type with Write() method satisfies Writer
type FileWriter struct{ f *os.File }
func (fw FileWriter) Write(p []byte) (int, error) { return fw.f.Write(p) }

var w Writer = FileWriter{} // works — implicit satisfaction
```

## Interface best practices

```go
// Accept interfaces, return structs (common idiom)
func NewProcessor(w io.Writer) *Processor { ... }

// Keep interfaces small — the io package is the model
type Reader interface { Read(p []byte) (n int, err error) }

// Define interfaces at the point of use, not the point of implementation
// Package consumer defines the interface; package provider implements it

// Empty interface (use any, not interface{})
func PrintAny(v any) { fmt.Println(v) }

// nil interface gotcha
var p *MyType = nil
var i any = p
fmt.Println(i == nil) // false! interface holds (type, nil) not nil
```

## Functional options pattern

Common for constructors with many optional parameters:

```go
type Server struct {
    addr    string
    timeout time.Duration
    maxConn int
}

type Option func(*Server)

func WithTimeout(d time.Duration) Option {
    return func(s *Server) { s.timeout = d }
}

func WithMaxConn(n int) Option {
    return func(s *Server) { s.maxConn = n }
}

func NewServer(addr string, opts ...Option) *Server {
    s := &Server{addr: addr, timeout: 30 * time.Second, maxConn: 100}
    for _, opt := range opts {
        opt(s)
    }
    return s
}

// Usage
srv := NewServer(":8080", WithTimeout(5*time.Second), WithMaxConn(50))
```
