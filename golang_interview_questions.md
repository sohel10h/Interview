# 🐹 Golang Interview Questions & Answers
> Covering ~80% of what you'll face in a real Go interview — from basics to advanced topics.

---

## 📚 Table of Contents

1. [Basics & Language Fundamentals](#1-basics--language-fundamentals)
2. [Data Types, Variables & Constants](#2-data-types-variables--constants)
3. [Functions & Methods](#3-functions--methods)
4. [Pointers](#4-pointers)
5. [Arrays, Slices & Maps](#5-arrays-slices--maps)
6. [Structs & Interfaces](#6-structs--interfaces)
7. [Concurrency — Goroutines & Channels](#7-concurrency--goroutines--channels)
8. [Error Handling](#8-error-handling)
9. [Memory Management & Garbage Collection](#9-memory-management--garbage-collection)
10. [Packages & Modules](#10-packages--modules)
11. [Advanced Topics](#11-advanced-topics)
12. [Coding Challenges](#12-coding-challenges)

---

## 1. Basics & Language Fundamentals

### Q1. What is Go (Golang)? Why was it created?
Go is an open-source, statically-typed, compiled programming language developed at Google by **Robert Griesemer, Rob Pike, and Ken Thompson**, released publicly in 2009. It was created to address challenges Google developers faced with large-scale, concurrent systems — aiming to combine the **safety of a typed language**, the **speed of C**, and the **readability of Python**.

Key goals:
- Fast compilation
- Efficient concurrency
- Simplicity and readability
- Strong standard library

---

### Q2. What are the key features of Go?
- **Goroutines** — lightweight, concurrent threads managed by the Go runtime
- **Channels** — safe communication mechanism between goroutines
- **Garbage Collection** — automatic memory management
- **Static Typing** with type inference (`:=` operator)
- **Fast Compilation** — compiles directly to machine code
- **No Classes/Inheritance** — uses structs and interfaces instead
- **Built-in Testing** via the `testing` package
- **Cross-platform** — statically linked binaries with no external runtime dependencies
- **Strong Standard Library**

---

### Q3. What is the Go workspace structure?
A Go workspace has three main directories:
```
workspace/
├── src/   → Contains Go source files organized by packages
├── pkg/   → Contains compiled package objects (.a files)
└── bin/   → Contains compiled/executable binaries
```

---

### Q4. What is the difference between `=` and `:=` in Go?
| Operator | Usage |
|----------|-------|
| `=`  | Assignment — variable must be **declared first** |
| `:=` | Short variable declaration — **declares and assigns** in one step; only inside functions |

```go
var x int = 10   // using =
y := 20          // using := (type inferred as int)
```

---

### Q5. What is the `init()` function in Go?
`init()` is a special function that runs **before `main()`** in a package. It is used to initialize application state (e.g., config loading, setting up DB connections).

Key facts:
- A package can have **multiple** `init()` functions
- They are called automatically — you cannot call them explicitly
- They run in the order they appear in the file
- They run **after all variable declarations** in the package

```go
func init() {
    fmt.Println("Package initialized")
}
```

---

### Q6. What is the `main` package in Go?
The `main` package is the **entry point** of a Go executable program. It must contain a `main()` function. Without it, Go cannot create an executable binary.

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

---

### Q7. Is Go case-sensitive?
**Yes.** Go is case-sensitive. `myVar` and `MyVar` are two different identifiers. Importantly:
- **Exported identifiers** (accessible outside the package) start with an **uppercase** letter — e.g., `fmt.Println`
- **Unexported identifiers** start with a **lowercase** letter — e.g., `myHelper()`

---

### Q8. What operators are used in Go?
- **Arithmetic:** `+`, `-`, `*`, `/`, `%`
- **Relational:** `==`, `!=`, `<`, `>`, `<=`, `>=`
- **Logical:** `&&`, `||`, `!`
- **Bitwise:** `&`, `|`, `^`, `<<`, `>>`
- **Assignment:** `=`, `+=`, `-=`, etc.
- **Pointer:** `*` (dereference), `&` (address-of)

---

## 2. Data Types, Variables & Constants

### Q9. What are the basic data types in Go?
| Category | Types |
|----------|-------|
| **Integer** | `int`, `int8`, `int16`, `int32`, `int64`, `uint`, `uint8`… |
| **Float** | `float32`, `float64` |
| **Complex** | `complex64`, `complex128` |
| **Boolean** | `bool` (`true` / `false`) |
| **String** | `string` |
| **Byte** | `byte` (alias for `uint8`) |
| **Rune** | `rune` (alias for `int32`, represents a Unicode code point) |

---

### Q10. What is a zero value in Go?
When a variable is declared but not initialized, Go assigns it a **zero value**:
| Type | Zero Value |
|------|-----------|
| `int` | `0` |
| `float64` | `0.0` |
| `bool` | `false` |
| `string` | `""` |
| `pointer` | `nil` |
| `slice`, `map`, `chan` | `nil` |

---

### Q11. What is the difference between `var` and `const`?
- `var` — declares a variable whose value can change
- `const` — declares a constant whose value is fixed at compile time

```go
var count int = 0     // can be changed
const Pi float64 = 3.14159  // cannot be changed
```

---

### Q12. What are string literals in Go?
There are two types:
- **Interpreted string literals** — enclosed in double quotes (`""`). Support escape sequences like `\n`, `\t`.
- **Raw string literals** — enclosed in backticks (`` ` ``). Taken literally, no escape processing. Great for multi-line strings and regex.

```go
interpreted := "Hello\nWorld"
raw := `Hello\nWorld`  // prints literally: Hello\nWorld
```

---

## 3. Functions & Methods

### Q13. How do you declare a function in Go?
```go
func functionName(param1 type1, param2 type2) returnType {
    // body
}
```

Example:
```go
func add(a int, b int) int {
    return a + b
}
```

---

### Q14. Can a Go function return multiple values?
**Yes!** This is one of Go's most praised features, commonly used for returning a result and an error.

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

---

### Q15. What is a variadic function?
A function that accepts a **variable number of arguments** using `...`:

```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

sum(1, 2, 3)      // works
sum(1, 2, 3, 4)   // also works
```

---

### Q16. What is a `defer` statement?
`defer` schedules a function call to run **just before the surrounding function returns**. Deferred calls execute in **LIFO (last-in, first-out)** order.

Common uses: closing files, releasing locks, cleanup tasks.

```go
func readFile() {
    f, _ := os.Open("file.txt")
    defer f.Close()  // guaranteed to run when readFile() exits
    // ... read file
}
```

---

### Q17. What are anonymous functions and closures?
- **Anonymous functions** have no name and can be assigned to variables or called immediately.
- **Closures** capture variables from their enclosing scope.

```go
add := func(x, y int) int {
    return x + y
}
fmt.Println(add(3, 4)) // 7

// Closure example
counter := func() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}()
fmt.Println(counter()) // 1
fmt.Println(counter()) // 2
```

---

## 4. Pointers

### Q18. What are pointers in Go?
A pointer stores the **memory address** of a variable. Go uses two pointer operators:
- `&` — address-of operator (returns the address of a variable)
- `*` — dereference operator (accesses the value at the address)

```go
x := 42
p := &x     // p holds the address of x
fmt.Println(*p) // 42 — dereference to get the value
*p = 100    // modifies x through the pointer
fmt.Println(x)  // 100
```

---

### Q19. What is the difference between `new()` and `make()`?
| | `new()` | `make()` |
|---|---------|---------|
| **Purpose** | Allocates memory, returns a pointer | Initializes and returns a reference |
| **Works with** | Any type | Only `slice`, `map`, `channel` |
| **Returns** | Pointer (`*T`) | The type itself (not a pointer) |
| **Initialization** | Zero value only | Fully initialized, ready to use |

```go
p := new(int)       // *int, zero value 0
s := make([]int, 5) // []int, length 5, capacity 5
```

---

## 5. Arrays, Slices & Maps

### Q20. What is the difference between an array and a slice in Go?
| | Array | Slice |
|--|-------|-------|
| **Size** | Fixed at compile time | Dynamic, can grow |
| **Type** | `[n]T` (e.g., `[5]int`) | `[]T` (e.g., `[]int`) |
| **Value/Reference** | Value type (copied) | Reference type (backed by array) |
| **Usage** | Rare in practice | Preferred in most cases |

```go
arr := [3]int{1, 2, 3}     // array
sl  := []int{1, 2, 3}      // slice
sl   = append(sl, 4)        // slice can grow
```

---

### Q21. How do you create a slice using `make()`?
```go
s := make([]int, length, capacity)

s := make([]int, 5)    // length=5, capacity=5
s := make([]int, 3, 10) // length=3, capacity=10
```

---

### Q22. How do you check if a slice is empty?
```go
if len(slice) == 0 {
    fmt.Println("Slice is empty")
}
```
> **Note:** Do not compare a slice to `nil` to check for emptiness — a non-nil slice can have length 0.

---

### Q23. What is a map in Go? How do you create and use one?
A map is a collection of **key-value pairs** (like a hash table / dictionary):

```go
// Create
m := map[string]int{
    "alice": 30,
    "bob":   25,
}

// Using make
m2 := make(map[string]int)

// Add/Update
m["charlie"] = 35

// Access
age := m["alice"]

// Check if key exists
val, ok := m["dave"]
if !ok {
    fmt.Println("key not found")
}

// Delete
delete(m, "bob")
```

---

## 6. Structs & Interfaces

### Q24. What is a struct in Go?
A `struct` is a composite data type that groups fields with different types under one name — similar to classes in OOP but without methods attached by default.

```go
type Person struct {
    Name string
    Age  int
}

p := Person{Name: "Alice", Age: 30}
fmt.Println(p.Name) // Alice
```

---

### Q25. Does Go support inheritance?
**No.** Go does not support classical inheritance. Instead, it uses **composition via struct embedding** to reuse behavior.

```go
type Animal struct {
    Name string
}

func (a Animal) Speak() string {
    return a.Name + " speaks"
}

type Dog struct {
    Animal     // embedding Animal
    Breed string
}

d := Dog{Animal: Animal{Name: "Rex"}, Breed: "Labrador"}
fmt.Println(d.Speak()) // Rex speaks — promoted method
```

---

### Q26. What is an interface in Go?
An interface defines a **set of method signatures** that a type must implement. Go interfaces are **implicitly satisfied** — no `implements` keyword needed.

```go
type Shape interface {
    Area() float64
    Perimeter() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64      { return r.Width * r.Height }
func (r Rectangle) Perimeter() float64 { return 2 * (r.Width + r.Height) }

// Rectangle implicitly implements Shape
var s Shape = Rectangle{Width: 5, Height: 3}
fmt.Println(s.Area()) // 15
```

---

### Q27. What is an empty interface (`interface{}`)?
`interface{}` (or `any` in Go 1.18+) accepts **any type** — it has no required methods.

```go
func printAnything(v interface{}) {
    fmt.Println(v)
}
printAnything(42)
printAnything("hello")
printAnything([]int{1, 2, 3})
```

---

### Q28. What is a type assertion?
A type assertion retrieves the underlying concrete type from an interface value:

```go
var i interface{} = "hello"

s, ok := i.(string)
if ok {
    fmt.Println(s) // hello
}
```

---

### Q29. What is a type switch?
A type switch allows branching based on the concrete type of an interface value:

```go
func describe(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Integer: %d\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}
```

---

## 7. Concurrency — Goroutines & Channels

### Q30. What is a goroutine?
A goroutine is a **lightweight thread of execution** managed by the Go runtime (not the OS). You launch one with the `go` keyword. Thousands can run concurrently with minimal overhead.

```go
func sayHello() {
    fmt.Println("Hello from goroutine!")
}

func main() {
    go sayHello()      // launches goroutine
    time.Sleep(1 * time.Second) // wait for it to finish
}
```

---

### Q31. What is the difference between concurrency and parallelism in Go?
- **Concurrency** — dealing with multiple tasks at the same time (may not run simultaneously). Go's goroutines enable this.
- **Parallelism** — executing multiple tasks **simultaneously** on multiple CPU cores. Go uses `runtime.GOMAXPROCS(n)` to control this.

> "Concurrency is about structure; parallelism is about execution." — Rob Pike

---

### Q32. What is a channel in Go?
Channels are **typed conduits** for communication between goroutines. They prevent data races by ensuring safe data passing.

```go
ch := make(chan int)

go func() {
    ch <- 42      // send
}()

val := <-ch       // receive
fmt.Println(val)  // 42
```

---

### Q33. What is the difference between buffered and unbuffered channels?
| | Unbuffered | Buffered |
|--|-----------|---------|
| **Capacity** | 0 | N (specified at creation) |
| **Sender blocks?** | Until receiver is ready | Only when buffer is full |
| **Receiver blocks?** | Until sender sends | Only when buffer is empty |
| **Sync** | Tightly synchronized | Loosely synchronized |

```go
unbuffered := make(chan int)       // unbuffered
buffered   := make(chan int, 5)    // buffered with capacity 5
```

---

### Q34. What is `select` in Go?
`select` waits on multiple channel operations — it's like a `switch` for channels. It picks the first case that is ready; if multiple are ready, it picks one at random.

```go
select {
case msg1 := <-ch1:
    fmt.Println("Received from ch1:", msg1)
case msg2 := <-ch2:
    fmt.Println("Received from ch2:", msg2)
case <-time.After(1 * time.Second):
    fmt.Println("Timeout!")
}
```

---

### Q35. What is a WaitGroup?
`sync.WaitGroup` is used to **wait for a collection of goroutines to finish**:

```go
var wg sync.WaitGroup

for i := 0; i < 5; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        fmt.Printf("Worker %d done\n", id)
    }(i)
}

wg.Wait() // blocks until all goroutines call Done()
```

---

### Q36. What is a Mutex? When would you use it?
`sync.Mutex` provides mutual exclusion — ensuring only one goroutine accesses a shared resource at a time:

```go
var mu sync.Mutex
var counter int

func increment() {
    mu.Lock()
    counter++
    mu.Unlock()
}
```

Use it when **multiple goroutines read/write shared memory** and you can't use channels instead.

---

### Q37. What is a race condition? How do you detect it in Go?
A **race condition** occurs when two goroutines access shared data concurrently and at least one writes to it.

Detect it using Go's built-in race detector:
```bash
go run -race main.go
go test -race ./...
```

---

## 8. Error Handling

### Q38. How does Go handle errors?
Go does not have exceptions. Instead, functions return an `error` value as their **last return value**. The caller must check it explicitly.

```go
result, err := someFunction()
if err != nil {
    log.Fatal(err)
}
```

---

### Q39. How do you create a custom error in Go?
**Option 1:** Using `errors.New()`:
```go
err := errors.New("something went wrong")
```

**Option 2:** Using `fmt.Errorf()` (supports formatting + wrapping):
```go
err := fmt.Errorf("failed to process user %d: %w", userID, originalErr)
```

**Option 3:** Custom error type (implement the `Error() string` interface):
```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on '%s': %s", e.Field, e.Message)
}
```

---

### Q40. What is `panic` and `recover` in Go?
- **`panic`** — stops normal execution, unwinds the stack, runs deferred functions, and crashes the program (unless recovered).
- **`recover`** — used inside a deferred function to **catch a panic** and resume normal execution.

```go
func safeDiv(a, b int) (result int, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("recovered from panic: %v", r)
        }
    }()
    return a / b, nil
}
```

> Use `panic` only for truly unrecoverable situations. Prefer returning errors for expected failure cases.

---

## 9. Memory Management & Garbage Collection

### Q41. How does Go manage memory?
Go uses **automatic garbage collection (GC)** — developers don't manually allocate or free memory. The GC runs concurrently to find and reclaim unreachable memory.

Key memory allocation functions:
- `new(T)` — allocates zeroed memory, returns `*T`
- `make(T, ...)` — allocates and initializes slices, maps, channels

---

### Q42. What is the Go garbage collector?
Go uses a **tri-color mark-and-sweep concurrent GC**. It runs in the background with minimal pause times. In recent Go versions, GC pauses are typically under 1 millisecond.

You can interact with it via:
```go
runtime.GC()        // manually trigger GC
runtime.GOMAXPROCS(n) // set number of OS threads
```

---

### Q43. What is a memory leak in Go and how do you prevent it?
Common Go memory leaks:
- Goroutines that never terminate (goroutine leaks)
- Unbounded caches or slices
- Unclosed resources (files, DB connections)

Prevention:
- Always close channels/files using `defer`
- Use `context.Context` to cancel goroutines
- Profile with `pprof`: `go tool pprof`

---

## 10. Packages & Modules

### Q44. What are packages in Go?
A **package** is a directory of `.go` files that belong to the same namespace. Every Go file starts with `package <name>`. Packages enable code organization and reuse.

```go
package mypackage
```

The **`main`** package is special — it defines an executable program.

---

### Q45. What is Go Modules (`go mod`)?
Go Modules are the official dependency management system (introduced in Go 1.11). Key files:
- `go.mod` — defines module name and dependencies
- `go.sum` — cryptographic checksums for dependencies

```bash
go mod init myapp       # initialize a new module
go get github.com/x/y   # add a dependency
go mod tidy             # clean up unused dependencies
```

---

### Q46. What is the difference between an exported and unexported identifier?
- **Exported** — starts with **uppercase**, accessible from other packages: `fmt.Println`, `http.Handler`
- **Unexported** — starts with **lowercase**, only accessible within the same package: `myHelper()`, `internalField`

---

## 11. Advanced Topics

### Q47. What is a goroutine leak?
A goroutine leak occurs when a goroutine is started but **never terminates**, causing it to stay in memory indefinitely. Common cause: blocking on a channel that nobody will send to.

```go
// LEAK: goroutine blocks forever on ch receive
ch := make(chan int)
go func() {
    val := <-ch  // nobody sends to ch — goroutine leaks!
    fmt.Println(val)
}()
```

**Fix:** Use `context.Context` for cancellation:
```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()
```

---

### Q48. What is the `context` package used for?
The `context` package carries **deadlines, cancellation signals, and request-scoped values** across goroutines and API boundaries.

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

result, err := fetchData(ctx)
```

---

### Q49. What is the difference between `Println` and `Printf` in Go?
| Function | Usage |
|----------|-------|
| `fmt.Println` | Prints with newline; no format string |
| `fmt.Printf` | Formatted output using format verbs (`%d`, `%s`, `%v`) |
| `fmt.Sprintf` | Returns a formatted string (doesn't print) |

---

### Q50. What are Go generics (introduced in Go 1.18)?
Generics allow writing **type-parametric functions and types** — functions that work with any type satisfying a constraint.

```go
func Map[T, U any](slice []T, f func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = f(v)
    }
    return result
}

doubled := Map([]int{1, 2, 3}, func(x int) int { return x * 2 })
// [2 4 6]
```

---

### Q51. What is struct embedding vs. composition?
Go achieves code reuse through **embedding** (not inheritance):
- **Embedding** promotes all fields and methods of the embedded type to the outer struct
- **Composition** means including a type as a named field

```go
// Embedding (fields/methods are promoted)
type Dog struct {
    Animal
}

// Composition (explicit access needed)
type Dog struct {
    animal Animal
}
d.animal.Speak() // must use explicit field name
```

---

### Q52. What is `iota` in Go?
`iota` is a special constant identifier in `const` blocks that **auto-increments** starting from 0:

```go
type Weekday int

const (
    Sunday Weekday = iota // 0
    Monday                // 1
    Tuesday               // 2
    Wednesday             // 3
)
```

---

### Q53. What is method receiver — value vs pointer?
```go
type Counter struct { count int }

// Value receiver — works on a copy
func (c Counter) Value() int { return c.count }

// Pointer receiver — modifies the original
func (c *Counter) Increment() { c.count++ }
```

Use **pointer receivers** when:
- You need to modify the struct
- The struct is large (avoids copying)

---

### Q54. How does Go handle multiple return values in assignment?
You can use the blank identifier `_` to discard values you don't need:

```go
result, _ := someFunc()   // discard the error (use with caution!)
_, err    := someFunc()   // discard the result
```

---

### Q55. What are build tags in Go?
Build tags (or build constraints) control which files are included in a build:

```go
//go:build linux
// +build linux   (older syntax)

package main
```

Useful for platform-specific code or test-only files.

---

## 12. Coding Challenges

### Q56. Reverse a string without using built-in functions
```go
func reverse(s string) string {
    runes := []rune(s)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}
```

---

### Q57. Check if a number is prime
```go
func isPrime(n int) bool {
    if n < 2 { return false }
    for i := 2; i*i <= n; i++ {
        if n%i == 0 { return false }
    }
    return true
}
```

---

### Q58. FizzBuzz in Go
```go
for i := 1; i <= 100; i++ {
    switch {
    case i%15 == 0:
        fmt.Println("FizzBuzz")
    case i%3 == 0:
        fmt.Println("Fizz")
    case i%5 == 0:
        fmt.Println("Buzz")
    default:
        fmt.Println(i)
    }
}
```

---

### Q59. Find duplicates in a slice
```go
func findDuplicates(nums []int) []int {
    seen := make(map[int]bool)
    var dups []int
    for _, n := range nums {
        if seen[n] {
            dups = append(dups, n)
        }
        seen[n] = true
    }
    return dups
}
```

---

### Q60. Fibonacci using goroutines and channels
```go
func fibonacci(n int, ch chan int) {
    a, b := 0, 1
    for i := 0; i < n; i++ {
        ch <- a
        a, b = b, a+b
    }
    close(ch)
}

func main() {
    ch := make(chan int, 10)
    go fibonacci(10, ch)
    for v := range ch {
        fmt.Println(v)
    }
}
```

---

## 🧠 Quick-Reference Cheat Sheet

| Topic | Key Points |
|-------|-----------|
| Goroutines | `go func(){}()` — lightweight, managed by runtime |
| Channels | `make(chan T)` — use for goroutine communication |
| Error handling | Always return `error` as last value; check `!= nil` |
| Interfaces | Implicitly satisfied; no `implements` keyword |
| Defer | Runs at function exit; LIFO order |
| Panic/Recover | For unrecoverable errors; recover inside defer |
| `new` vs `make` | `new` → pointer to zero value; `make` → initialized slice/map/chan |
| Struct embedding | Go's way to compose/reuse types (not inheritance) |
| Blank identifier `_` | Discard unwanted return values |
| Exported identifiers | Start with uppercase letter |

---

## 📌 Tips for the Interview

1. **Know concurrency deeply** — goroutines, channels, select, WaitGroup, Mutex are almost always asked.
2. **Understand interfaces** — Go's type system is interface-driven; be ready for practical examples.
3. **Error handling philosophy** — explain why Go prefers explicit error returns over exceptions.
4. **Be able to write code** — reverse strings, fibonacci, detect race conditions live.
5. **Know the difference** — array vs slice, buffered vs unbuffered channels, new vs make.
6. **Context package** — increasingly common in senior/backend Go interviews.
7. **Generics (1.18+)** — understand the basics, especially for newer roles.

---

*Last updated: April 2026 | Based on aggregated interview data from Turing, InterviewBit, roadmap.sh, Educative, and community sources.*
