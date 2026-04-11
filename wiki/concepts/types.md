---
title: Types
type: concept
tags: [types, slices, maps, pointers, arrays]
created: 2026-04-11
updated: 2026-04-11
sources: []
---

Go's type system: primitives, composite types, and type definitions. See also: [[Structs and Interfaces]], [[Generics]].

## Primitive types

```go
// Integers
var i8  int8   = 127          // -128 to 127
var i16 int16  = 32767
var i32 int32  = 2147483647
var i64 int64  = 9223372036854775807
var i   int    = 42           // platform word size (64-bit on modern systems)
var u   uint   = 42

// Floats
var f32 float32 = 3.14
var f64 float64 = 3.141592653589793

// Complex
var c complex128 = 1 + 2i

// Byte and rune
var b byte = 'A'   // alias for uint8
var r rune = '🦫'  // alias for int32, Unicode code point

// String — immutable sequence of bytes (UTF-8)
s := "Hello, 世界"
fmt.Println(len(s))         // byte count, not character count
fmt.Println(len([]rune(s))) // character count
```

## Slices

Slices are the primary dynamic collection — backed by an array, with length and capacity.

```go
// Creation
var s []int              // nil slice, len=0, cap=0
s = []int{1, 2, 3}      // literal
s = make([]int, 5)       // len=5, cap=5, all zeros
s = make([]int, 0, 10)   // len=0, cap=10 — pre-allocated

// Append
s = append(s, 4)
s = append(s, 5, 6, 7)
s = append(s, other...)  // spread another slice

// Slicing (shares underlying array!)
a := []int{0, 1, 2, 3, 4}
b := a[1:3]  // [1 2], len=2, cap=4
b[0] = 99   // modifies a too!

// Clone to avoid aliasing (Go 1.21+)
import "slices"
c := slices.Clone(a)

// Common patterns
// Delete element at index i (order-preserving)
s = append(s[:i], s[i+1:]...)

// Delete element at index i (fast, unordered)
s[i] = s[len(s)-1]
s = s[:len(s)-1]

// Filter in-place (Go 1.23+)
s = slices.DeleteFunc(s, func(v int) bool { return v < 0 })

// Sort
slices.Sort(s)
slices.SortFunc(items, func(a, b Item) int {
    return cmp.Compare(a.Name, b.Name)
})

// Search (requires sorted slice)
idx, found := slices.BinarySearch(s, 42)
```

## Maps

```go
// Creation
var m map[string]int   // nil map — reads ok, writes panic
m = map[string]int{}  // empty map
m = make(map[string]int)
m = map[string]int{"a": 1, "b": 2}

// Read — always returns zero value if key absent
v := m["missing"]  // 0, no panic

// Read with existence check
v, ok := m["key"]
if !ok { /* key not present */ }

// Write / delete
m["key"] = 42
delete(m, "key")

// Iterate (random order)
for k, v := range m {
    fmt.Println(k, v)
}

// maps package (Go 1.21+)
import "maps"
keys := slices.Collect(maps.Keys(m))   // all keys as slice
maps.Clone(m)                           // shallow copy
maps.DeleteFunc(m, func(k string, v int) bool { return v == 0 })
```

## Pointers

```go
x := 42
p := &x        // pointer to x
fmt.Println(*p) // dereference: 42
*p = 100       // modify through pointer
fmt.Println(x)  // 100

// new — allocate zero value, return pointer
p2 := new(int)  // *int pointing to 0

// Pointers enable mutation across function calls
func increment(n *int) { *n++ }
increment(&x)

// Go has NO pointer arithmetic — safe by design
```

## Arrays

Fixed-size, value types (rarely used directly — prefer slices).

```go
var a [5]int             // [0 0 0 0 0]
a := [3]string{"a","b","c"}
a := [...]int{1, 2, 3}  // compiler infers size

// Arrays are compared by value
[3]int{1,2,3} == [3]int{1,2,3} // true
```

## Type definitions and aliases

```go
// Type definition — new distinct type
type Celsius float64
type Fahrenheit float64
// Celsius(100) + Fahrenheit(32) → compile error: prevents accidental mixing

func (c Celsius) ToFahrenheit() Fahrenheit {
    return Fahrenheit(c*9/5 + 32)
}

// Type alias — same type, different name (used for API migration)
type MyInt = int  // MyInt IS int, not a new type
```

## any and type assertions

```go
// any is an alias for interface{} (Go 1.18+)
var v any = "hello"

// Type assertion
s, ok := v.(string)  // safe: ok=true, s="hello"
s = v.(string)       // panics if wrong type

// Use type switch for multiple types (see [[Language Basics]])
```
