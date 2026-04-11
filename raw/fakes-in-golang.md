In golang, you **can** create fakes (or test doubles) for things like `math/rand.Intn` (or the newer `math/rand/v2.IntN`), but Go's design makes it **not as straightforward** as in Java/Kotlin with Mockito or PowerMock.

Go has **no built-in mocking framework** and **no way to dynamically replace package-level functions** at runtime (no monkey-patching without unsafe hacks, which are discouraged in production code).

Here are the idiomatic approaches, ranked from most recommended to less common:

### 1. Best Practice: Dependency Injection via Interface (Recommended for experienced engineers)

Wrap the random behavior behind a small interface. This is the cleanest, most testable, and most "Go-like" way.

```go
// random.go  (or wherever makes sense)
type Randomizer interface {
    IntN(n int) int   // Use IntN if you're on math/rand/v2
    // Add other methods you need: Float64(), etc.
}

// Production implementation
type defaultRandomizer struct{}

func (defaultRandomizer) IntN(n int) int {
    return rand.IntN(n)  // or rand.Intn(n) for older math/rand
}

// In your main code, inject it
type MyService struct {
    rng Randomizer
}

func NewMyService(rng Randomizer) *MyService {
    if rng == nil {
        rng = defaultRandomizer{}
    }
    return &MyService{rng: rng}
}

func (s *MyService) SomeMethod() {
    value := s.rng.IntN(100)
    // ...
}
```

**In tests:**

```go
type fakeRandomizer struct {
    nextIntN func(n int) int
}

func (f fakeRandomizer) IntN(n int) int {
    return f.nextIntN(n)
}

func TestMyService_SomeMethod(t *testing.T) {
    fake := fakeRandomizer{
        nextIntN: func(n int) int { return 42 },  // deterministic
    }
    svc := NewMyService(fake)

    // Now your test is fully deterministic
    result := svc.SomeMethod()
    // assertions...
}
```

This pattern scales well. Many Go codebases (including large ones) use tiny interfaces exactly like this for `io.Reader`, `http.RoundTripper`, databases, etc.

### 2. Function Type Injection (Lightweight Alternative)

If you only need one or two random calls, you can inject a plain function:

```go
type MyService struct {
    randIntN func(int) int
}

func NewMyService(randIntN func(int) int) *MyService {
    if randIntN == nil {
        randIntN = rand.IntN  // or rand.Intn
    }
    return &MyService{randIntN: randIntN}
}
```

In tests:

```go
svc := NewMyService(func(n int) int { return 7 })
```

This is very common and feels lightweight.

### 3. Seeding for Determinism (When You Don't Want to Change Design Much)

If the randomness is not core to the logic you're testing, just create a seeded `*rand.Rand` (or `rand/v2.Rand`) and pass it around, or set a fixed seed.

```go
r := rand.New(rand.NewPCG(42, 123))  // deterministic sequence
value := r.IntN(100)
```

This works great for property-based or fuzz-style testing too.

### 4. What Doesn't Work Well

- Directly mocking `math/rand.Intn` or replacing the global generator in tests → not reliable or clean.
- Using third-party mock libraries (testify/mock, etc.) for this → overkill and less idiomatic for stdlib functions.
- Global variable tricks or `init()` hacks → fragile, especially with concurrent tests (`-race` will hate it).

### Quick Advice from a Java/Kotlin Perspective

In Java you'd reach for Mockito to stub `Random.nextInt()`.  
In Go, the philosophy is: **make the dependency explicit** (via interface or func) instead of hiding it.

This feels like more upfront work the first few times, but it leads to:
- Better separation of concerns
- Easier testing (no reflection magic)
- More composable code

Once you internalize this, you'll apply the same pattern to HTTP clients, DB connections, clocks (`time.Now`), etc.

**Note on math/rand vs math/rand/v2**:  
Go 1.22+ strongly encourages `math/rand/v2` (cleaner API, better performance). Use `IntN`, `UintN`, etc. The same DI patterns apply.

Would you like me to show a complete small example (e.g., a service that picks a random item from a slice), or compare it to how you'd do the equivalent in Kotlin with mocking? Or do you have a specific piece of code you're trying to test?Yes — since you already use the clean functional-injection style in Java (no mock frameworks, just swapping implementations via `Supplier` / functional interfaces), the Go equivalent is almost identical in spirit and even lighter-weight.

Here’s a minimal, side-by-side example that directly mirrors `math/rand.IntN` (or the older `math/rand.Intn`).

### Java version (using `IntUnaryOperator`)
This is the exact pattern you described — injecting a functional supplier-style provider for random values.

```java
import java.util.Objects;
import java.util.function.IntUnaryOperator;
import java.util.concurrent.ThreadLocalRandom;

public class GameService {

    private final IntUnaryOperator randomIntN;   // like Random.nextInt(bound)

    public GameService(IntUnaryOperator randomIntN) {
        // graceful default to real randomness
        this.randomIntN = Objects.requireNonNullElse(randomIntN,
                n -> ThreadLocalRandom.current().nextInt(n));
    }

    /**
     * Returns a random int in the range [0, n)
     */
    public int pickRandom(int n) {
        if (n <= 0) {
            throw new IllegalArgumentException("n must be positive");
        }
        return randomIntN.applyAsInt(n);
    }
}
```

**Production usage:**
```java
GameService service = new GameService(null);        // uses real randomness
int roll = service.pickRandom(6);                   // e.g. 0-5 like a die
```

**Test usage (deterministic fake):**
```java
GameService service = new GameService(n -> 42);     // always returns 42
int result = service.pickRandom(100);               // guaranteed 42
```

### Go equivalent (using a function type)
Go’s idiomatic equivalent to your Java `IntUnaryOperator` is simply a function type. This is extremely common and zero-overhead.

```go
// game.go
package main

import "math/rand/v2"   // preferred in Go 1.22+

type GameService struct {
    rand func(int) int   // exactly equivalent to your IntUnaryOperator
}

func NewGameService(r func(int) int) *GameService {
    if r == nil {
        r = rand.IntN       // real randomness (math/rand/v2)
    }
    return &GameService{rand: r}
}

func (s *GameService) PickRandom(n int) int {
    if n <= 0 {
        panic("n must be positive") // or return an error — your choice
    }
    return s.rand(n)
}
```

**Production usage:**
```go
service := NewGameService(nil)      // uses real randomness
roll := service.PickRandom(6)       // e.g. 0-5
```

**Test usage (deterministic fake):**
```go
service := NewGameService(func(n int) int {
    return 42                       // always 42, no matter what n is
})
result := service.PickRandom(100)   // guaranteed 42
```

### Why this feels natural coming from Java/Kotlin
- The Go `func(int) int` is the direct analog of your `IntUnaryOperator` / `Supplier`-style injection.
- No extra libraries, no reflection, no boilerplate.
- You can name the field whatever you like (`rng`, `random`, `randIntN`, etc.).
- If you prefer a more Java-like interface instead of a bare function type, you can do that too (shown below), but most Go codebases prefer the lighter function type for single-method cases like this.

#### Alternative Go version with a small interface (if you prefer it)
```go
type RandomIntN interface {
    IntN(n int) int
}

type GameService struct {
    rand RandomIntN
}

func NewGameService(r RandomIntN) *GameService {
    if r == nil {
        r = defaultRandom{} // implement IntN with rand.IntN
    }
    return &GameService{rand: r}
}
```

The function-type version is what you’ll see most often in real Go codebases for exactly this use case.

That’s the complete mechanics. Drop this into a small project and you’ll be swapping randomness exactly the way you’re used to. Want me to expand it into a full tiny working program (with tests) or show the same pattern for something else (e.g. `time.Now`, HTTP client, etc.)? Just say the word.
