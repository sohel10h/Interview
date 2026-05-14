# 🐹 Go Interview Questions for Senior .NET / C# Developers
> You know C# deeply. This guide maps everything you know in .NET to how Go does it — and highlights the traps interviewers will set for you.

**Your profile:** 10 years C# / .NET → 2 years Go → applying for a **Senior Go role**

Interviewers will ask: *"How would you do X from .NET in Go?"* — This file answers every one of those questions.

---

## 📚 Table of Contents

1. [Philosophy & Mindset Shift](#1-philosophy--mindset-shift)
2. [OOP in Go — No Classes, No Inheritance](#2-oop-in-go--no-classes-no-inheritance)
3. [Interfaces — Implicit vs Explicit](#3-interfaces--implicit-vs-explicit)
4. [Error Handling — Exceptions vs Error Values](#4-error-handling--exceptions-vs-error-values)
5. [Concurrency — async/await & TPL vs Goroutines & Channels](#5-concurrency--asyncawait--tpl-vs-goroutines--channels)
6. [Generics — C# Generics vs Go Generics](#6-generics--c-generics-vs-go-generics)
7. [Memory Management & Garbage Collection](#7-memory-management--garbage-collection)
8. [Null vs nil — Null Safety](#8-null-vs-nil--null-safety)
9. [Collections — LINQ vs Go Idioms](#9-collections--linq-vs-go-idioms)
10. [Dependency Injection & IoC](#10-dependency-injection--ioc)
11. [Testing — xUnit/NUnit vs Go's testing Package](#11-testing--xunitnunit-vs-gos-testing-package)
12. [Web Development — ASP.NET vs net/http](#12-web-development--aspnet-vs-nethttp)
13. [Type System Differences](#13-type-system-differences)
14. [Design Patterns in Go](#14-design-patterns-in-go)
15. [Tooling & Ecosystem Comparison](#15-tooling--ecosystem-comparison)
16. [Senior-Level Architecture Questions](#16-senior-level-architecture-questions)
17. [Tricky "Gotcha" Questions for .NET Devs](#17-tricky-gotcha-questions-for-net-devs)

---

## 1. Philosophy & Mindset Shift

### Q1. What is the biggest mental shift moving from C# to Go?

This is almost always the **first question** for a C# developer in a Go interview. Be ready with a complete, confident answer.

| C# / .NET Mindset | Go Mindset |
|---|---|
| "Inheritance is the foundation of reuse" | "Composition over inheritance — always" |
| "Exceptions handle errors" | "Errors are just values — check them explicitly" |
| "async/await makes concurrency readable" | "Goroutines make concurrency *cheap* — spawn thousands" |
| "Rich OOP hierarchy" | "Flat structs + interfaces — no class trees" |
| "Frameworks do the heavy lifting" | "Prefer stdlib — less is more" |
| "Null means absence of value" | "Zero values mean absence — avoid nil where possible" |
| "Dependency Injection containers are standard" | "Manual DI via function arguments and interfaces" |

**What to say in the interview:**
> "The hardest shift for me was unlearning exception-based error handling. In Go, you can't throw and forget — every error must be explicitly handled at the call site. It felt tedious at first, but it makes call stacks dramatically easier to trace in production."

---

### Q2. Why does Go not have `try/catch`?

Go's designers believe that **exceptions create hidden control flow** — a function may secretly throw, sending control somewhere unexpected. By making errors explicit return values:
- Every failure path is visible at the call site
- No surprise propagation up the stack
- Code reviews make error handling obvious

```go
// In Go — error is part of the function contract
result, err := riskyOperation()
if err != nil {
    return fmt.Errorf("context: %w", err)
}
```

```csharp
// In C# — exception propagates invisibly
try {
    var result = RiskyOperation();
} catch (SomeException ex) {
    // Caught somewhere far from the source
}
```

---

### Q3. Why does Go not have a `class` keyword?

Go's creators deliberately excluded classes because:
- Classes encourage deep inheritance trees → tight coupling
- Go favors **behavior-based design** via interfaces
- Struct + methods achieve everything a class does, more explicitly
- No "magic" base class behavior (no `Object.ToString()` override surprises)

---

## 2. OOP in Go — No Classes, No Inheritance

### Q4. How do you achieve encapsulation in Go without classes?

In C# you use `private`, `protected`, `public` on class members. In Go, encapsulation is at the **package level**:

- **Uppercase** first letter → exported (public to other packages)
- **Lowercase** first letter → unexported (private to the package)

```go
// In package "user"
type User struct {
    Name  string  // exported — accessible outside package
    email string  // unexported — only accessible within "user" package
}

func (u *User) Email() string { return u.email }        // getter
func (u *User) SetEmail(e string) { u.email = e }       // setter
```

```csharp
// C# equivalent
public class User {
    public string Name { get; set; }
    private string _email;
    public string Email { get => _email; set => _email = value; }
}
```

**Key difference:** Go has no `protected` — you can't expose something to a subclass but not the world. Go avoids this because there are no subclasses.

---

### Q5. How do you implement inheritance in Go?

**You don't.** Go uses **struct embedding** (composition) instead. An embedded struct's methods and fields are *promoted* to the outer struct.

```go
// Go — struct embedding (composition)
type Animal struct {
    Name string
}
func (a Animal) Breathe() string { return a.Name + " breathes" }

type Dog struct {
    Animal              // embedded — promotes Name and Breathe()
    Breed string
}

d := Dog{Animal: Animal{Name: "Rex"}, Breed: "Lab"}
d.Breathe()  // promoted — works without d.Animal.Breathe()
d.Name       // promoted field
```

```csharp
// C# — classical inheritance
class Animal {
    public string Name { get; set; }
    public virtual string Breathe() => $"{Name} breathes";
}
class Dog : Animal {
    public string Breed { get; set; }
}
```

**Interview-critical difference:** In Go, embedding is **not inheritance**. `Dog` does not "is-a" `Animal`. You cannot pass a `Dog` where an `Animal` is expected. You can pass it where an interface satisfied by `Animal`'s methods is expected.

---

### Q6. How do you simulate abstract classes in Go?

Go has no abstract classes. The Go equivalent is an **interface** — define the contract, have concrete structs implement it.

```go
// Go — interface as abstract contract
type Shape interface {
    Area() float64
    Draw()
}

type Circle struct{ Radius float64 }
func (c Circle) Area() float64 { return math.Pi * c.Radius * c.Radius }
func (c Circle) Draw()          { fmt.Println("Drawing circle") }
```

```csharp
// C# — abstract class
abstract class Shape {
    public abstract double Area();
    public abstract void Draw();
}
class Circle : Shape {
    public double Radius { get; set; }
    public override double Area() => Math.PI * Radius * Radius;
    public override void Draw() => Console.WriteLine("Drawing circle");
}
```

---

### Q7. How does method overriding work in Go?

Go has **no virtual methods or overriding**. If an embedded struct and an outer struct both define the same method, the **outer struct's method takes precedence** (called method shadowing).

```go
type Base struct{}
func (b Base) Hello() { fmt.Println("Base Hello") }

type Child struct{ Base }
func (c Child) Hello() { fmt.Println("Child Hello") }  // shadows Base.Hello()

c := Child{}
c.Hello()       // "Child Hello"
c.Base.Hello()  // "Base Hello" — explicit access still works
```

There is no `override` keyword, no `virtual`, no `base.Method()` — only explicit field access.

---

## 3. Interfaces — Implicit vs Explicit

### Q8. What is the biggest difference between Go interfaces and C# interfaces?

**Go interfaces are satisfied implicitly** — no `implements` keyword, no explicit declaration. If a type has all the methods the interface requires, it satisfies it automatically.

```go
// Go — implicit satisfaction
type Writer interface {
    Write(p []byte) (n int, err error)
}

type MyWriter struct{}
func (w MyWriter) Write(p []byte) (int, error) { ... }

// MyWriter automatically satisfies Writer — no declaration needed
var w Writer = MyWriter{}
```

```csharp
// C# — explicit implementation required
interface IWriter {
    int Write(byte[] p);
}
class MyWriter : IWriter {         // must explicitly declare
    public int Write(byte[] p) { ... }
}
```

**Why this matters:** In Go, you can define an interface *after* a type exists, and if the type happens to match, it satisfies it. This is called **structural typing** (duck typing with compile-time verification).

---

### Q9. What is the `io.Reader` / `io.Writer` pattern and why is it so important in Go?

These are Go's most important interfaces — equivalent to `Stream` in .NET:

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

Any type that implements `Read` can be used wherever `io.Reader` is expected — files, HTTP bodies, strings, buffers, network connections. This is Go's polymorphism at its finest.

```csharp
// C# equivalent — System.IO.Stream
Stream stream = new FileStream("file.txt", FileMode.Open);
stream = new MemoryStream(); // or this
stream = response.GetResponseStream(); // or this
```

In Go:
```go
var r io.Reader
r = os.Open("file.txt")     // *os.File implements io.Reader
r = bytes.NewReader(data)   // *bytes.Reader implements io.Reader
r = resp.Body               // http.Response.Body implements io.Reader
```

---

### Q10. Can a Go struct implement multiple interfaces?

**Yes** — and it's the Go way of achieving what C# does with multiple interface implementation.

```go
type ReadWriter interface {
    Reader
    Writer
}

// MyBuffer implements both Read and Write — so it satisfies ReadWriter
type MyBuffer struct { data []byte }
func (b *MyBuffer) Read(p []byte) (int, error)  { ... }
func (b *MyBuffer) Write(p []byte) (int, error) { ... }
```

---

## 4. Error Handling — Exceptions vs Error Values

### Q11. How do you translate C# exception patterns to Go?

| C# Pattern | Go Equivalent |
|---|---|
| `throw new Exception(msg)` | `return errors.New(msg)` |
| `throw new ArgumentException(msg)` | `return fmt.Errorf("invalid arg: %w", err)` |
| `try/catch` | `if err != nil { ... }` |
| `finally` | `defer` |
| `catch (Exception ex) when (ex is X)` | Type assertion: `var e *MyErr; errors.As(err, &e)` |
| `ex.InnerException` | `errors.Unwrap(err)` |
| Custom exception class | Custom struct implementing `error` interface |
| Re-throw: `throw;` | `return fmt.Errorf("context: %w", err)` (wraps with `%w`) |

---

### Q12. How do you implement custom error types in Go (like custom C# exceptions)?

```go
// Go — custom error type (equivalent to a custom C# exception)
type DatabaseError struct {
    Code    int
    Message string
    Query   string
}

func (e *DatabaseError) Error() string {
    return fmt.Sprintf("DB error %d: %s (query: %s)", e.Code, e.Message, e.Query)
}

// Usage
func queryDB(sql string) error {
    return &DatabaseError{Code: 500, Message: "timeout", Query: sql}
}

// Checking the type — equivalent to C#'s "catch (DatabaseException ex)"
err := queryDB("SELECT ...")
var dbErr *DatabaseError
if errors.As(err, &dbErr) {
    fmt.Println("DB Code:", dbErr.Code)
}
```

```csharp
// C# equivalent
public class DatabaseException : Exception {
    public int Code { get; }
    public string Query { get; }
    public DatabaseException(int code, string msg, string query)
        : base(msg) { Code = code; Query = query; }
}
try {
    QueryDB("SELECT ...");
} catch (DatabaseException ex) {
    Console.WriteLine($"DB Code: {ex.Code}");
}
```

---

### Q13. What is error wrapping in Go and how does it compare to InnerException?

Go 1.13 introduced `%w` for error wrapping — equivalent to C#'s `InnerException`:

```go
// Wrap an error with context
err := fmt.Errorf("service layer failed: %w", originalErr)

// Unwrap — like ex.InnerException
unwrapped := errors.Unwrap(err)

// Check the chain — like "is ex or any inner ex a DatabaseError?"
errors.Is(err, io.EOF)          // checks entire chain
errors.As(err, &targetType)     // finds first matching type in chain
```

```csharp
// C# equivalent
throw new ServiceException("service layer failed", innerException: originalEx);
// Check inner: ex.InnerException is DatabaseException
```

---

### Q14. When should you use `panic` in Go? Is it like `throw`?

**No — `panic` is NOT a replacement for `throw`.** In Go:
- `panic` = truly unrecoverable situations (like `Environment.FailFast()` in .NET)
- Use regular `error` returns for all expected/recoverable failure cases
- `panic` cascades up the call stack, running all `defer`s, then crashes

Use `panic` for:
- Programming errors (nil pointer dereference — Go does this automatically)
- Failed assertions during startup (e.g., missing config)
- Index out of bounds

```go
// WRONG — don't use panic like throw
func divide(a, b int) int {
    if b == 0 {
        panic("division by zero") // ❌ use error return instead
    }
    return a / b
}

// CORRECT
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero") // ✅
    }
    return a / b, nil
}
```

---

## 5. Concurrency — async/await & TPL vs Goroutines & Channels

### Q15. What is the fundamental difference between C# async/await and Go goroutines?

This is one of the **most asked** questions for C# developers going to Go interviews.

| | C# async/await | Go goroutines |
|--|----------------|---------------|
| **Model** | State machine (compiler transform) | Real lightweight thread (green thread) |
| **Keyword** | `async` / `await` | `go` |
| **Cost** | ~1KB object per Task | ~2–8KB stack (grows dynamically) |
| **Scalability** | ~50K+ concurrent tasks | 100K–1M+ goroutines |
| **Return type** | `Task<T>` / `Task` | No return — use channels |
| **Cancellation** | `CancellationToken` | `context.Context` |
| **Philosophy** | "Communicate by sharing memory" | "Share memory by communicating" |
| **Viral?** | Yes — `async` spreads through call chain | No — any function can be a goroutine |

**The async virus problem:** In C#, once you go async, every caller must also be async. In Go, there's no such distinction — you just call `go myFunc()`.

```csharp
// C# — async is viral
public async Task<string> GetDataAsync() {
    var result = await httpClient.GetStringAsync(url);
    return result;
}
// Every caller of GetDataAsync must also be async...
```

```go
// Go — no viral spreading
func getData() string {
    resp, _ := http.Get(url)
    body, _ := io.ReadAll(resp.Body)
    return string(body)
}

// Run concurrently — caller doesn't change at all
go getData()
```

---

### Q16. How do you replace `Task.WhenAll()` in Go?

C#'s `Task.WhenAll()` runs multiple tasks and waits for all to complete. In Go, use `sync.WaitGroup`:

```csharp
// C#
var tasks = new[] {
    FetchUserAsync(1),
    FetchUserAsync(2),
    FetchUserAsync(3),
};
var results = await Task.WhenAll(tasks);
```

```go
// Go — WaitGroup equivalent
var wg sync.WaitGroup
results := make([]User, 3)

for i := 0; i < 3; i++ {
    wg.Add(1)
    go func(idx int) {
        defer wg.Done()
        results[idx] = fetchUser(idx + 1)
    }(i)
}
wg.Wait()
```

Or using channels for result collection:
```go
ch := make(chan User, 3)
for i := 1; i <= 3; i++ {
    go func(id int) { ch <- fetchUser(id) }(i)
}
// Collect all results
users := make([]User, 0, 3)
for i := 0; i < 3; i++ {
    users = append(users, <-ch)
}
```

---

### Q17. How do you replace `CancellationToken` in Go?

Go's `context.Context` is the equivalent of `CancellationToken` in .NET — and it does more (carries deadlines and request-scoped values too):

```csharp
// C#
public async Task DoWorkAsync(CancellationToken ct) {
    await Task.Delay(1000, ct);
    ct.ThrowIfCancellationRequested();
}
var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
await DoWorkAsync(cts.Token);
```

```go
// Go
func doWork(ctx context.Context) error {
    select {
    case <-time.After(1 * time.Second):
        return nil
    case <-ctx.Done():
        return ctx.Err() // context.Canceled or context.DeadlineExceeded
    }
}

ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
err := doWork(ctx)
```

**Convention:** `ctx` is always the **first parameter** in any Go function that may block, call external services, or run in a goroutine. This is idiomatic Go.

---

### Q18. How do you replace `Task.WhenAny()` / first-to-complete in Go?

Use `select` with multiple channels:

```csharp
// C#
var winner = await Task.WhenAny(task1, task2, task3);
```

```go
// Go — select picks the first ready channel
ch1 := make(chan string, 1)
ch2 := make(chan string, 1)

go func() { ch1 <- slowOperation1() }()
go func() { ch2 <- slowOperation2() }()

select {
case result := <-ch1:
    fmt.Println("ch1 won:", result)
case result := <-ch2:
    fmt.Println("ch2 won:", result)
case <-time.After(5 * time.Second):
    fmt.Println("timeout")
}
```

---

### Q19. What replaces `lock` / `Monitor` in Go?

```csharp
// C#
private readonly object _lock = new();
lock (_lock) {
    _counter++;
}
```

```go
// Go — sync.Mutex
var mu sync.Mutex
var counter int

mu.Lock()
counter++
mu.Unlock()

// Or idiomatically with defer
func increment() {
    mu.Lock()
    defer mu.Unlock()
    counter++
}
```

For read-heavy scenarios (like C#'s `ReaderWriterLockSlim`):
```go
var rwmu sync.RWMutex

// Multiple readers allowed
rwmu.RLock()
val := sharedData
rwmu.RUnlock()

// Exclusive write
rwmu.Lock()
sharedData = newVal
rwmu.Unlock()
```

---

### Q20. What is the Go equivalent of `Parallel.ForEach`?

```csharp
// C#
Parallel.ForEach(items, item => ProcessItem(item));
```

```go
// Go — goroutine per item with WaitGroup
var wg sync.WaitGroup
for _, item := range items {
    wg.Add(1)
    go func(i Item) {
        defer wg.Done()
        processItem(i)
    }(item)
}
wg.Wait()
```

For bounded parallelism (like setting `MaxDegreeOfParallelism` in C#):
```go
semaphore := make(chan struct{}, 10) // max 10 concurrent goroutines
var wg sync.WaitGroup
for _, item := range items {
    wg.Add(1)
    semaphore <- struct{}{}  // acquire
    go func(i Item) {
        defer wg.Done()
        defer func() { <-semaphore }()  // release
        processItem(i)
    }(item)
}
wg.Wait()
```

---

## 6. Generics — C# Generics vs Go Generics

### Q21. How do Go generics compare to C# generics?

Go added generics in **Go 1.18 (2022)** — much later than C# (2005). They share the same concept but have different syntax and constraints.

| | C# Generics | Go Generics |
|--|-------------|-------------|
| **Syntax** | `List<T>` | `[]T` (type parameter) |
| **Constraints** | `where T : IComparable<T>` | `[T constraints.Ordered]` |
| **Any type** | `where T : class` or unconstrained | `[T any]` |
| **Multiple** | `<T, U>` | `[T, U any]` |
| **Maturity** | Very mature, rich features | Newer — no generic methods on types yet |

```go
// Go generic function
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Go generic with constraint
func Max[T constraints.Ordered](a, b T) T {
    if a > b { return a }
    return b
}
```

```csharp
// C# equivalent
public static List<U> Map<T, U>(List<T> list, Func<T, U> fn) =>
    list.Select(fn).ToList();

public static T Max<T>(T a, T b) where T : IComparable<T> =>
    a.CompareTo(b) > 0 ? a : b;
```

**Current Go generic limitation:** You cannot define generic methods on a type (only generic functions and types). This is a known gap vs. C#.

---

## 7. Memory Management & Garbage Collection

### Q22. How does Go's GC compare to .NET's GC?

| | .NET GC | Go GC |
|--|---------|--------|
| **Type** | Generational (Gen0/1/2) | Tri-color concurrent mark-and-sweep |
| **Pause time** | Sub-millisecond (recent .NET) | Sub-millisecond (Go 1.14+) |
| **LOH** | Large Object Heap for >85KB | No separate heap — but large allocations go to heap |
| **Tuning** | `GCSettings.LatencyMode`, `GC.Collect()` | `GOGC` env var, `runtime/debug.SetGCPercent()` |
| **Finalizers** | `~Destructor()` / `IDisposable` | `runtime.SetFinalizer()` (rare, discouraged) |
| **`IDisposable`** | `using` statement | `defer f.Close()` |

---

### Q23. What replaces `IDisposable` and `using` in Go?

Go has no `IDisposable`. Use `defer` to guarantee resource cleanup:

```csharp
// C#
using (var conn = new SqlConnection(connStr)) {
    conn.Open();
    // conn.Dispose() guaranteed even on exception
}
// Or with C# 8+
using var conn = new SqlConnection(connStr);
```

```go
// Go
conn, err := sql.Open("postgres", connStr)
if err != nil { return err }
defer conn.Close() // guaranteed to run when function returns

// File
f, err := os.Open("file.txt")
if err != nil { return err }
defer f.Close()  // guaranteed cleanup
```

---

### Q24. What is the Go equivalent of `stackalloc` / value types on the stack?

Go has **escape analysis** — the compiler decides if a variable lives on the stack or heap:
- Small variables used only locally → stack (fast, no GC pressure)
- Variables that escape scope (returned pointer, stored in interface, captured by closure) → heap

```go
// Stack allocated — s doesn't escape
s := MyStruct{x: 1}
fmt.Println(s)

// Heap allocated — pointer escapes
s := &MyStruct{x: 1}
return s  // escapes to heap
```

Check with: `go build -gcflags="-m" ./...` — shows escape analysis decisions.

---

## 8. Null vs nil — Null Safety

### Q25. How does Go handle null compared to C#?

| | C# | Go |
|--|----|----|
| **Null keyword** | `null` | `nil` |
| **Null for value types** | Requires `Nullable<T>` / `int?` | Zero value — no nullable needed |
| **Null for reference types** | Default for all reference types | Pointer types, interfaces, slices, maps, channels |
| **Null safety** | C# 8+ nullable reference types (`string?`) | No built-in null safety — be careful with interfaces |
| **Null check** | `obj == null` / `obj is null` | `ptr == nil` / `if val == nil` |

```go
// In Go — zero values replace most nullable scenarios
var count int      // 0, not null
var name string    // "", not null
var ptr *MyStruct  // nil — be careful!

// Avoid returning nil interfaces — a gotcha!
// This is NOT nil even though the value is:
var err *MyError = nil
var iface error = err  // iface != nil !!! — known Go gotcha
```

---

### Q26. What is the dangerous nil interface gotcha in Go?

A classic trap interviewers use for C# devs moving to Go:

```go
type MyError struct { msg string }
func (e *MyError) Error() string { return e.msg }

func mayFail(fail bool) error {
    var err *MyError
    if fail {
        err = &MyError{"something broke"}
    }
    return err  // ❌ DANGEROUS! Returns a non-nil interface holding a nil pointer
}

err := mayFail(false)
if err != nil {
    fmt.Println("error:", err)  // This WILL print, even though the error is "nil"!
}
```

**Fix:** Return `nil` directly:
```go
func mayFail(fail bool) error {
    if fail {
        return &MyError{"something broke"}
    }
    return nil  // ✅ Correct
}
```

---

## 9. Collections — LINQ vs Go Idioms

### Q27. What replaces LINQ in Go?

Go has **no built-in LINQ equivalent** before Go 1.18. You write loops manually, or use generics (1.18+) and third-party packages like `golang.org/x/exp/slices`.

| LINQ Operation | Go Equivalent |
|---|---|
| `.Where(x => x > 5)` | Manual loop / `slices.DeleteFunc` |
| `.Select(x => x * 2)` | Manual loop / generic `Map` function |
| `.FirstOrDefault()` | Loop with early return |
| `.Any(x => ...)` | Loop with early return |
| `.OrderBy(x => x.Name)` | `sort.Slice(s, func(i,j) bool {...})` |
| `.GroupBy(x => x.Key)` | Loop building `map[K][]V` |
| `.ToDictionary(x => x.Id)` | Loop building `map[K]V` |

```csharp
// C# LINQ
var adults = people
    .Where(p => p.Age >= 18)
    .Select(p => p.Name)
    .OrderBy(n => n)
    .ToList();
```

```go
// Go equivalent
var adults []string
for _, p := range people {
    if p.Age >= 18 {
        adults = append(adults, p.Name)
    }
}
sort.Strings(adults)
```

With Go 1.21+ `slices` and `maps` packages:
```go
import "slices"
adults := slices.Collect(
    func(yield func(string) bool) {
        for _, p := range people {
            if p.Age >= 18 {
                if !yield(p.Name) { return }
            }
        }
    })
slices.Sort(adults)
```

---

### Q28. What replaces `List<T>`, `Dictionary<K,V>`, `HashSet<T>` in Go?

| C# Collection | Go Equivalent |
|---|---|
| `List<T>` | `[]T` (slice) |
| `Dictionary<K,V>` | `map[K]V` |
| `HashSet<T>` | `map[T]struct{}` |
| `Queue<T>` | Channel or slice |
| `Stack<T>` | Slice with append/pop idiom |
| `LinkedList<T>` | `container/list` package |
| `IEnumerable<T>` | `[]T` slice or channel or iterator func |
| `ConcurrentDictionary<K,V>` | `sync.Map` |
| `ImmutableList<T>` | No built-in; convention + care |

```go
// HashSet equivalent in Go
seen := make(map[string]struct{})
seen["alice"] = struct{}{}
_, exists := seen["alice"]  // exists = true

// ConcurrentDictionary equivalent
var m sync.Map
m.Store("key", "value")
val, ok := m.Load("key")
```

---

## 10. Dependency Injection & IoC

### Q29. How do you do Dependency Injection in Go without a DI container?

.NET developers are used to `IServiceCollection`, `AddScoped()`, `AddSingleton()`, etc. **Go has no DI framework in its standard library** — and idiomatic Go rarely uses one.

**Go's approach: Constructor Injection via function parameters and interfaces**

```go
// Define dependencies as interfaces
type UserRepository interface {
    FindByID(id int) (*User, error)
}

type EmailService interface {
    Send(to, subject, body string) error
}

// Service depends on interfaces, not concrete types
type UserService struct {
    repo  UserRepository
    email EmailService
}

// Constructor (manual DI)
func NewUserService(repo UserRepository, email EmailService) *UserService {
    return &UserService{repo: repo, email: email}
}

// Wire manually in main()
func main() {
    repo  := database.NewUserRepository(db)
    email := smtp.NewEmailService(smtpConfig)
    svc   := NewUserService(repo, email)
}
```

```csharp
// C# equivalent with ASP.NET DI
services.AddScoped<IUserRepository, UserRepository>();
services.AddSingleton<IEmailService, SmtpEmailService>();
services.AddScoped<UserService>();
```

If a DI framework is needed in Go, popular options: `google/wire` (compile-time), `uber/dig` or `uber/fx` (runtime).

---

### Q30. How do you handle singleton pattern in Go?

```go
// Go singleton using sync.Once
type Config struct{ DSN string }
var (
    instance *Config
    once     sync.Once
)

func GetConfig() *Config {
    once.Do(func() {
        instance = &Config{DSN: os.Getenv("DATABASE_URL")}
    })
    return instance
}
```

```csharp
// C# equivalent
public sealed class Config {
    private static readonly Lazy<Config> _instance =
        new(() => new Config());
    public static Config Instance => _instance.Value;
}
```

---

## 11. Testing — xUnit/NUnit vs Go's testing Package

### Q31. How does Go testing compare to xUnit/NUnit/MSTest?

| | C# (xUnit/NUnit) | Go |
|--|------------------|----|
| **Test file** | `*.Test.cs` or any class | `*_test.go` |
| **Test func naming** | `[Fact]` attribute | Must start with `Test` |
| **Assertions** | `Assert.Equal()` | Manual `t.Errorf()` or third-party `testify` |
| **Setup/Teardown** | `[SetUp]` / `IDisposable` | `TestMain()` / `t.Cleanup()` |
| **Subtests** | `[Theory]` + `[InlineData]` | `t.Run("name", func...)` |
| **Mocking** | Moq, NSubstitute | Manual interface mocks or `testify/mock` |
| **Coverage** | dotnet test --collect | `go test -cover -coverprofile=c.out` |
| **Benchmarks** | BenchmarkDotNet | Built-in `Benchmark*` functions |

```go
// Go test
func TestAdd(t *testing.T) {
    got := add(2, 3)
    want := 5
    if got != want {
        t.Errorf("add(2,3) = %d; want %d", got, want)
    }
}

// Table-driven tests — Go's equivalent of [Theory] + [InlineData]
func TestAddTable(t *testing.T) {
    tests := []struct {
        name string
        a, b int
        want int
    }{
        {"positive", 2, 3, 5},
        {"negative", -1, -2, -3},
        {"zero", 0, 0, 0},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := add(tt.a, tt.b); got != tt.want {
                t.Errorf("add(%d,%d) = %d; want %d", tt.a, tt.b, got, tt.want)
            }
        })
    }
}

// Benchmark — like BenchmarkDotNet
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        add(2, 3)
    }
}
```

---

### Q32. How do you mock interfaces in Go?

Go has no Moq/NSubstitute. You write minimal manual mocks or use `testify/mock`:

```go
// Manual mock — idiomatic Go
type MockUserRepo struct {
    users map[int]*User
}
func (m *MockUserRepo) FindByID(id int) (*User, error) {
    if u, ok := m.users[id]; ok { return u, nil }
    return nil, fmt.Errorf("user %d not found", id)
}

// Test using the mock
func TestUserService_GetUser(t *testing.T) {
    repo := &MockUserRepo{users: map[int]*User{1: {Name: "Alice"}}}
    svc  := NewUserService(repo, nil)
    user, err := svc.GetUser(1)
    // assert...
}
```

---

## 12. Web Development — ASP.NET vs net/http

### Q33. How does Go web development compare to ASP.NET?

| | ASP.NET Core | Go (net/http + Gin) |
|--|--------------|---------------------|
| **Server** | Kestrel (built-in) | `net/http` (built-in) |
| **Routing** | Attribute routing / minimal API | `http.ServeMux` / Gin / Chi |
| **Middleware** | `IMiddleware` / `Use()` pipeline | Handler chain / `http.Handler` wrapping |
| **Model binding** | Automatic from JSON/form | Manual `json.NewDecoder(r.Body).Decode()` |
| **Validation** | `[Required]`, FluentValidation | Manual or `go-playground/validator` |
| **Dependency Injection** | Built-in container | Constructor injection |
| **ORM** | Entity Framework Core | GORM / `database/sql` |
| **Configuration** | `appsettings.json` + `IConfiguration` | `os.Getenv()`, Viper, or custom |
| **Auth** | ASP.NET Identity, JWT Bearer | Manual JWT or third-party |

```go
// Go minimal HTTP server
http.HandleFunc("/api/users", func(w http.ResponseWriter, r *http.Request) {
    users := []User{{ID: 1, Name: "Alice"}}
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)
})
http.ListenAndServe(":8080", nil)
```

```csharp
// C# minimal API
app.MapGet("/api/users", () => new[] { new User { Id = 1, Name = "Alice" } });
app.Run();
```

---

### Q34. How do you write middleware in Go vs ASP.NET?

```go
// Go middleware — wraps http.Handler
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)  // call the next handler
        log.Printf("%s %s %v", r.Method, r.URL.Path, time.Since(start))
    })
}

// Apply
http.Handle("/api/", loggingMiddleware(apiHandler))
```

```csharp
// C# middleware
app.Use(async (context, next) => {
    var start = DateTime.UtcNow;
    await next.Invoke();
    var elapsed = DateTime.UtcNow - start;
    logger.LogInformation($"{context.Request.Method} {context.Request.Path} {elapsed.TotalMs}ms");
});
```

---

## 13. Type System Differences

### Q35. How does Go handle type conversion vs C# casting?

Go requires **explicit type conversions** — no implicit widening/narrowing conversions (except untyped constants):

```go
var i int = 42
var f float64 = float64(i)   // explicit — like C# (float)i
var u uint = uint(f)

// String conversions
s := strconv.Itoa(42)       // int to string (NOT string(42) — that gives a Unicode char!)
n, _ := strconv.Atoi("42") // string to int
```

```csharp
int i = 42;
double f = (double)i;        // explicit cast
double f2 = i;               // or implicit widening — NOT in Go!
string s = i.ToString();
int n = int.Parse("42");
```

---

### Q36. Does Go have `enum`?

**No.** Go uses `const` + `iota` to simulate enums:

```go
type Status int

const (
    StatusPending  Status = iota  // 0
    StatusActive                  // 1
    StatusInactive                // 2
    StatusDeleted                 // 3
)

func (s Status) String() string {
    return [...]string{"Pending", "Active", "Inactive", "Deleted"}[s]
}
```

```csharp
public enum Status { Pending, Active, Inactive, Deleted }
```

**Limitation:** Go's iota enums don't have built-in string conversion, exhaustiveness checking, or flag support — you implement those yourself.

---

### Q37. Does Go have `struct` inheritance through `base` type like C#?

No. But you can embed and call the embedded struct's methods explicitly:

```go
type Base struct{ ID int }
func (b *Base) Save() error { return saveToDb(b.ID) }

type User struct {
    Base
    Name string
}

u := User{Base: Base{ID: 1}, Name: "Alice"}
u.Save()       // calls Base.Save() — promoted
u.Base.Save()  // explicit call — also works
```

There's no `base.Method()` syntax. To call the embedded struct's version of a shadowed method, use explicit field access.

---

## 14. Design Patterns in Go

### Q38. How do common Gang of Four patterns translate to Go?

#### Repository Pattern
```go
type UserRepo interface {
    FindByID(ctx context.Context, id int) (*User, error)
    Save(ctx context.Context, u *User) error
    Delete(ctx context.Context, id int) error
}

type postgresUserRepo struct{ db *sql.DB }
func (r *postgresUserRepo) FindByID(ctx context.Context, id int) (*User, error) { ... }
```

#### Options Pattern (replaces Builder / named parameters)
```go
type ServerConfig struct {
    host    string
    port    int
    timeout time.Duration
}

type Option func(*ServerConfig)

func WithHost(h string) Option    { return func(c *ServerConfig) { c.host = h } }
func WithPort(p int) Option       { return func(c *ServerConfig) { c.port = p } }
func WithTimeout(t time.Duration) Option { return func(c *ServerConfig) { c.timeout = t } }

func NewServer(opts ...Option) *Server {
    cfg := &ServerConfig{host: "localhost", port: 8080, timeout: 30 * time.Second}
    for _, o := range opts { o(cfg) }
    return &Server{cfg: cfg}
}

// Usage — like named parameters
srv := NewServer(WithPort(9090), WithTimeout(60*time.Second))
```

#### Observer / Event Pattern
```go
// Go channels as event bus
type EventBus struct {
    subscribers map[string][]chan Event
    mu          sync.RWMutex
}

func (b *EventBus) Subscribe(topic string) <-chan Event {
    ch := make(chan Event, 10)
    b.mu.Lock()
    b.subscribers[topic] = append(b.subscribers[topic], ch)
    b.mu.Unlock()
    return ch
}

func (b *EventBus) Publish(topic string, e Event) {
    b.mu.RLock()
    for _, ch := range b.subscribers[topic] { ch <- e }
    b.mu.RUnlock()
}
```

---

## 15. Tooling & Ecosystem Comparison

### Q39. What are the Go equivalents of .NET CLI tools?

| .NET Tool | Go Equivalent |
|---|---|
| `dotnet build` | `go build ./...` |
| `dotnet run` | `go run main.go` |
| `dotnet test` | `go test ./...` |
| `dotnet publish` | `go build -o app` (single binary, no publish folder) |
| `dotnet add package` | `go get github.com/pkg/name` |
| `NuGet` | Go modules (`go.mod`) |
| `dotnet format` | `gofmt -w .` or `goimports -w .` |
| `dotnet tool install --global` | `go install github.com/...@latest` |
| `dotnet-counters` / `dotnet-trace` | `go tool pprof`, `go tool trace` |
| `BenchmarkDotNet` | Built-in `testing.B` + `go test -bench=.` |
| ReSharper / Rider analysis | `go vet ./...`, `staticcheck` |
| `swagger` / Swashbuckle | `swaggo/swag` |

---

### Q40. What is `gofmt` and why does it matter?

`gofmt` is Go's **mandatory code formatter**. Unlike C# (where formatting is a style choice), Go code must be formatted with `gofmt`. There's one correct way to format Go code — no debates.

```bash
gofmt -w .          # format all .go files in place
goimports -w .      # gofmt + auto-manage imports
go vet ./...        # static analysis (like Roslyn analyzers)
staticcheck ./...   # advanced linter (like ReSharper)
```

This is checked in CI in most Go shops — if your code isn't `gofmt`'d, the PR is rejected.

---

## 16. Senior-Level Architecture Questions

### Q41. How would you design a microservice in Go vs ASP.NET?

**In .NET:** You'd reach for ASP.NET Core with built-in DI, middleware pipeline, Entity Framework, Serilog, and deploy as a Docker container.

**In Go, the idiomatic approach:**
```
cmd/
  server/
    main.go         ← wire everything, minimal code
internal/
  domain/           ← business logic, pure Go structs
  repository/       ← data access (interfaces)
  service/          ← business logic (interfaces)
  handler/          ← HTTP handlers (thin layer)
  middleware/       ← auth, logging, recovery
pkg/
  config/           ← configuration loading
  logger/           ← structured logging (zerolog/zap)
```

Key principles in Go microservices:
- **No global state** — pass dependencies explicitly
- **Thin HTTP handlers** — handlers call services, not business logic directly
- **Interfaces at boundaries** — makes testing easy without DI containers
- **Context everywhere** — cancellation and timeouts at every layer

---

### Q42. How do you implement structured logging in Go? What replaces Serilog?

```go
// Using zerolog (popular Serilog replacement)
import "github.com/rs/zerolog/log"

log.Info().
    Str("user_id", "123").
    Str("action", "login").
    Dur("latency", elapsed).
    Msg("user logged in")

// Using uber-go/zap (high-performance)
import "go.uber.org/zap"

logger, _ := zap.NewProduction()
logger.Info("user logged in",
    zap.String("user_id", "123"),
    zap.Duration("latency", elapsed),
)
```

```csharp
// C# Serilog equivalent
Log.Information("User {UserId} logged in after {Latency}ms", "123", elapsed.TotalMs);
```

---

### Q43. How does Go handle database access compared to Entity Framework?

Go has **no ORM in stdlib**. The standard is `database/sql` with a driver:

```go
// Raw SQL — like Dapper in .NET
db, _ := sql.Open("postgres", connStr)

var user User
err := db.QueryRowContext(ctx,
    "SELECT id, name, email FROM users WHERE id = $1", id,
).Scan(&user.ID, &user.Name, &user.Email)

// Bulk query
rows, _ := db.QueryContext(ctx, "SELECT id, name FROM users WHERE active = $1", true)
defer rows.Close()
for rows.Next() {
    var u User
    rows.Scan(&u.ID, &u.Name)
    users = append(users, u)
}
```

Popular Go ORMs/query builders:
- **GORM** — closest to EF Core (active record style)
- **sqlx** — light extension of `database/sql` (like Dapper)
- **sqlc** — generates type-safe Go from SQL (closest to compiled EF migrations)
- **Bun** — modern ORM with good performance

---

## 17. Tricky "Gotcha" Questions for .NET Devs

### Q44. What happens when you range over a slice and modify it?

```go
s := []int{1, 2, 3}
for i, v := range s {
    s = append(s, v*10)  // ❌ Modifying slice during range
    _ = i
}
// range captures the original slice header — won't loop infinitely
// but appended elements are NOT visited
```

---

### Q45. Why does this goroutine loop have a bug? (Classic C# closure trap — worse in Go)

```go
// BUG — all goroutines capture the same 'i' variable
for i := 0; i < 5; i++ {
    go func() {
        fmt.Println(i)  // ❌ All likely print 5
    }()
}

// FIX — pass i as argument
for i := 0; i < 5; i++ {
    go func(n int) {
        fmt.Println(n)  // ✅ Each goroutine gets its own copy
    }(i)
}
```

> **Note:** Go 1.22 fixed loop variable capture — each loop iteration now has its own variable. But know this for older codebases.

---

### Q46. What is the difference between a `nil` slice and an empty slice?

```go
var nilSlice []int         // nil slice   — len=0, cap=0, nil==true
emptySlice := []int{}     // empty slice — len=0, cap=0, nil==false
emptySlice2 := make([]int, 0) // also empty, not nil

// Both behave the same for append and range
// BUT json.Marshal treats them differently!
json.Marshal(nilSlice)    // → null
json.Marshal(emptySlice)  // → []
```

This is a **common interview trap** — and a real production bug for C# devs who don't know it.

---

### Q47. What does Go do with unused imports and variables?

**Go refuses to compile** if you have:
- An unused imported package
- An unused local variable

```go
import "fmt"  // ❌ compile error if fmt is never used

x := 5       // ❌ compile error if x is never used
```

Use blank identifier `_` to explicitly discard:
```go
import _ "github.com/lib/pq"  // side-effect only import (registers DB driver)
_ = someFunc()                // explicitly discard return value
```

This is very different from C# where unused variables are just warnings.

---

### Q48. Why can't you compare structs directly in Go sometimes?

Structs are **comparable** only if all their fields are comparable. Slices, maps, and functions are not comparable:

```go
type Good struct{ Name string; Age int }   // comparable — string and int are comparable
type Bad  struct{ Tags []string }           // not comparable — slice is not comparable

g1, g2 := Good{"Alice", 30}, Good{"Alice", 30}
fmt.Println(g1 == g2) // true ✅

b1, b2 := Bad{[]string{"a"}}, Bad{[]string{"a"}}
fmt.Println(b1 == b2) // ❌ compile error: cannot compare
// Use reflect.DeepEqual() or write custom Equal() method
```

---

## 🗂️ Master Comparison Summary

| Concept | C# / .NET | Go |
|---|---|---|
| Classes | `class` keyword | `struct` + methods |
| Inheritance | `: BaseClass` | Struct embedding |
| Interfaces | Explicit `implements` | Implicit structural typing |
| Error handling | `try/catch/throw` | Return `error` value |
| Async | `async/await`, `Task<T>` | Goroutines + channels |
| Cancellation | `CancellationToken` | `context.Context` |
| Null | `null`, nullable `?` | `nil`, zero values |
| LINQ | `.Where().Select().OrderBy()` | Manual loops / generics |
| DI | `IServiceCollection` + container | Constructor injection via interfaces |
| Dispose | `IDisposable` + `using` | `defer f.Close()` |
| Enums | `enum` keyword | `const` + `iota` |
| Generics | `List<T>` since C# 2.0 | `[T any]` since Go 1.18 |
| Events | `event`, `delegate` | Channels / callbacks |
| Reflection | `System.Reflection` | `reflect` package (discouraged) |
| Testing | xUnit/NUnit + attributes | `*_test.go` + `t.Run` |
| Benchmarks | BenchmarkDotNet | Built-in `testing.B` |
| Package manager | NuGet / `dotnet add` | Go modules (`go mod`) |
| Formatter | EditorConfig / ReSharper | `gofmt` (mandatory, one standard) |
| Logging | Serilog / NLog | `zerolog`, `zap`, `slog` (stdlib 1.21) |

---

## 🎯 Interview Strategy for .NET → Go Senior Role

1. **Lead with trade-offs, not superiority** — say "Go trades X for Y" not "Go is better than C#"
2. **Show you understand Go's philosophy** — mention "errors as values", "composition over inheritance", "share memory by communicating"
3. **Acknowledge the gaps** — Go generics are newer, no LINQ is verbose, no DI container is manual work. Interviewers respect honesty.
4. **Leverage your .NET depth** — use it to explain concepts clearly: "This is like `CancellationToken` but also carries deadlines and values"
5. **Know the gotchas** — nil interface, closure capture, nil vs empty slice JSON difference — these show real Go experience
6. **Mention the toolchain** — `gofmt`, `go vet`, `pprof`, `go test -race` — shows production-level maturity

---

*Last updated: April 2026 | Tailored for .NET/C# developers interviewing for senior Go roles.*
