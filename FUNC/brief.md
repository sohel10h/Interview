# Func Brief for C# and .NET Core

## What `Func` Is

`Func` is a built-in generic delegate in C# that represents a method which returns a value.

Simple interview line:

"`Func` is a predefined delegate used when you want to pass a method or lambda that returns a value."

Examples:

```csharp
Func<int> getNumber = () => 100;
Func<int, int> square = x => x * x;
Func<int, int, int> add = (a, b) => a + b;
```

Rule to remember:

- The last type parameter is always the return type
- All earlier type parameters are input parameters

So:

- `Func<int>` means no input, returns `int`
- `Func<string, int>` means input `string`, returns `int`
- `Func<int, int, int>` means inputs `int, int`, returns `int`

## Why `Func` Matters

`Func` is heavily used in modern C# because it works naturally with:

- lambda expressions
- LINQ
- callbacks
- strategy-style logic
- deferred execution
- filtering, mapping, and transformation

In real .NET code, you will often see `Func` instead of creating a custom delegate type.

## Basic Example

```csharp
Func<int, int, int> multiply = (a, b) => a * b;
int result = multiply(4, 5);

Console.WriteLine(result); // 20
```

## When to Use `Func`

Use `Func` when:

- you need to pass behavior as a parameter
- that behavior returns a value
- the delegate is simple and does not need a domain-specific name
- you are writing LINQ-style transformation or calculation code

Do not prefer `Func` by default when:

- the delegate meaning is business-specific and deserves a clear name
- readability is better with a custom delegate
- you need an event, where event-handler conventions matter more

## `delegate` vs `Func`

Both represent method references. `Func` is just a built-in generic delegate type, while `delegate` lets you define your own delegate type.

## Comparison Table

| Topic | Custom `delegate` | `Func` |
|---|---|---|
| Definition | User-defined delegate type | Built-in generic delegate |
| Return value | Can return value or `void` depending on declaration | Always returns a value |
| Naming | You choose a meaningful name | Uses standard `Func<...>` syntax |
| Readability | Better when the behavior has domain meaning | Better for short general-purpose callbacks |
| Boilerplate | More code | Less code |
| Common usage | Events, domain-specific callbacks, expressive APIs | LINQ, transformations, calculators, selectors |
| Lambda support | Yes | Yes |
| Interview example | `public delegate decimal TaxCalculator(Order order);` | `Func<Order, decimal>` |

## When to Use `Func` vs Custom Delegate

Use `Func` when:

- the signature is simple
- the code is general-purpose
- you want compact lambda-based code

Use a custom delegate when:

- the name improves intent
- the callback is part of the domain language
- the API is public and readability matters

Example:

```csharp
public delegate decimal TaxCalculator(Order order);
```

This may read better than:

```csharp
Func<Order, decimal>
```

if the callback has strong business meaning.

## Problem Solving Example 1: Pass Calculation Logic

```csharp
public static int Execute(int a, int b, Func<int, int, int> operation)
{
    return operation(a, b);
}
```

Usage:

```csharp
int sum = Execute(10, 20, (x, y) => x + y);
int max = Execute(10, 20, (x, y) => x > y ? x : y);
```

What this shows:

- behavior can be passed dynamically
- one method supports multiple operations
- `Func` reduces custom delegate boilerplate

## Problem Solving Example 2: Build a Generic Mapper

This is a practical interview example.

```csharp
public static List<TResult> Map<TSource, TResult>(
    IEnumerable<TSource> items,
    Func<TSource, TResult> selector)
{
    List<TResult> result = new();

    foreach (var item in items)
    {
        result.Add(selector(item));
    }

    return result;
}
```

Usage:

```csharp
var numbers = new[] { 1, 2, 3, 4 };
var squares = Map(numbers, x => x * x);
var texts = Map(numbers, x => $"Value: {x}");
```

Why interviewers like this:

- very close to LINQ `Select`
- shows generic method + `Func`
- demonstrates reusable transformation logic

## Problem Solving Example 3: Delayed Execution

`Func` is often used to delay work until it is needed.

```csharp
public static T ExecuteWithLogging<T>(Func<T> operation)
{
    Console.WriteLine("Starting operation...");
    T result = operation();
    Console.WriteLine("Operation completed.");
    return result;
}
```

Usage:

```csharp
int value = ExecuteWithLogging(() => 50 * 2);
```

This is common in retry logic, caching, wrappers, and logging utilities.

## Hard Problem: Build a Retry Helper with `Func`

This is a stronger interview example because it shows `Func` used in real backend utility code.

```csharp
public static T Retry<T>(Func<T> operation, int maxAttempts)
{
    Exception? lastException = null;

    for (int attempt = 1; attempt <= maxAttempts; attempt++)
    {
        try
        {
            return operation();
        }
        catch (Exception ex)
        {
            lastException = ex;
        }
    }

    throw lastException ?? new InvalidOperationException("Operation failed.");
}
```

Usage:

```csharp
int result = Retry(() =>
{
    if (DateTime.UtcNow.Ticks % 2 == 0)
        throw new Exception("Temporary failure");

    return 42;
}, 3);
```

Why this is hard:

- uses `Func<T>` for deferred execution
- captures retryable logic cleanly
- very close to real service-layer utility code

## Hard Problem: Compose Multiple `Func`s

This is another strong interview problem because it tests function composition.

```csharp
public static Func<TInput, TOutput> Compose<TInput, TMiddle, TOutput>(
    Func<TInput, TMiddle> first,
    Func<TMiddle, TOutput> second)
{
    return input => second(first(input));
}
```

Usage:

```csharp
Func<string, string> trim = text => text.Trim();
Func<string, int> length = text => text.Length;

var trimmedLength = Compose(trim, length);
int result = trimmedLength("   hello   "); // 5
```

Why this matters:

- shows higher-order programming
- shows how methods can return functions
- useful for pipeline-style logic

## `Func` with LINQ

LINQ often uses `Func`.

Examples:

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };

var evenNumbers = numbers.Where(x => x % 2 == 0);
var doubled = numbers.Select(x => x * 2);
```

The lambdas passed to `Where` and `Select` are usually represented by delegate types such as `Func<T, bool>` and `Func<T, TResult>`.

## `Func` vs `Action` vs `Predicate`

- `Func`: returns a value
- `Action`: returns `void`
- `Predicate<T>`: returns `bool` for one input

Examples:

```csharp
Func<int, int> square = x => x * x;
Action<string> print = text => Console.WriteLine(text);
Predicate<int> isEven = x => x % 2 == 0;
```

## Common Interview Mistakes

- saying `Func` is different from delegates completely
- forgetting that `Func` is also a delegate
- forgetting that the last type argument is the return type
- using `Func` everywhere even when a named custom delegate is clearer
- confusing `Func<T, bool>` with `Predicate<T>`

## Quick Interview Summary

If asked for a short answer:

"`Func` is a built-in generic delegate in C# used for methods or lambdas that return a value. It is widely used in LINQ, callbacks, pipelines, and helper methods because it reduces boilerplate compared to custom delegates. I use `Func` when I need a simple reusable callback with a return value, and I use a custom delegate when a domain-specific name improves readability."

## References

- [Microsoft Learn - Func<TResult> Delegate](https://learn.microsoft.com/en-us/dotnet/api/system.func-1?view=net-10.0)
- [Microsoft Learn - Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [Microsoft Learn - Generic Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-delegates)
- [Microsoft Learn - Lambda expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions)
