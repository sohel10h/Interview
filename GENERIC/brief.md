# Generics Brief for C# and .NET Core

## What Generics Are

Generics let you write classes, methods, interfaces, and delegates without fixing the data type in advance.

Instead of writing separate code for `int`, `string`, `Product`, and `Order`, you write one reusable version using a type parameter such as `T`.

Example:

```csharp
public class Box<T>
{
    public T Value { get; }

    public Box(T value)
    {
        Value = value;
    }
}
```

Usage:

```csharp
var intBox = new Box<int>(10);
var textBox = new Box<string>("hello");
```

## Why Generics Matter

Main benefits:

- Type safety at compile time
- Code reuse
- Better readability than using `object`
- Better performance for value types because boxing and unboxing can be avoided

Interview line:

"Generics help us write reusable and type-safe code. They reduce casting errors and improve performance compared to non-generic `object`-based code."

## Generic Class, Method, Interface

Generic class:

```csharp
public class Repository<T>
{
    private readonly List<T> _items = new();

    public void Add(T item) => _items.Add(item);
    public IReadOnlyList<T> GetAll() => _items;
}
```

Generic method:

```csharp
public static T Echo<T>(T value)
{
    return value;
}
```

Generic interface:

```csharp
public interface IPrinter<T>
{
    void Print(T value);
}
```

## Generic Constraints

Constraints tell the compiler what a type parameter must support.

Common constraints:

- `where T : class`
- `where T : struct`
- `where T : new()`
- `where T : BaseClass`
- `where T : IComparable<T>`

Example:

```csharp
public static T Max<T>(T left, T right) where T : IComparable<T>
{
    return left.CompareTo(right) >= 0 ? left : right;
}
```

Why this matters:

Without the constraint, the compiler does not know whether `T` supports `CompareTo`.

## Generics vs object

Without generics:

```csharp
ArrayList items = new ArrayList();
items.Add(10);
items.Add("hello");
```

Problem:

- Mixed types can create runtime errors
- Casting is required
- Value types may be boxed

With generics:

```csharp
List<int> numbers = new List<int> { 10, 20, 30 };
```

This is safer and faster.

## Variance

This is a common interview topic for experienced roles.

- Covariance with `out`: lets you use a more derived type
- Contravariance with `in`: lets you use a less derived type

Example:

```csharp
IEnumerable<string> names = new List<string> { "A", "B" };
IEnumerable<object> objects = names;
```

Important:

- Variance works for some interfaces and delegates
- It applies to reference types
- Classes like `List<T>` are invariant

So this is not allowed:

```csharp
// List<object> items = new List<string>(); // invalid
```

## Problem Solving Examples

## Problem 1: Write a Generic Swap Method

```csharp
public static void Swap<T>(ref T left, ref T right)
{
    T temp = left;
    left = right;
    right = temp;
}
```

Usage:

```csharp
int a = 5, b = 10;
Swap(ref a, ref b);

string x = "first", y = "second";
Swap(ref x, ref y);
```

Interview point:

One method works for many data types without duplication.

## Problem 2: Find the Largest Element in a Generic Array

```csharp
public static T FindLargest<T>(IEnumerable<T> items) where T : IComparable<T>
{
    using var enumerator = items.GetEnumerator();

    if (!enumerator.MoveNext())
        throw new InvalidOperationException("Sequence cannot be empty.");

    T largest = enumerator.Current;

    while (enumerator.MoveNext())
    {
        if (enumerator.Current.CompareTo(largest) > 0)
            largest = enumerator.Current;
    }

    return largest;
}
```

Usage:

```csharp
var maxNumber = FindLargest(new[] { 3, 9, 2, 7 });
var maxText = FindLargest(new[] { "apple", "pear", "banana" });
```

Why this is good:

- Shows generic constraints
- Shows problem solving
- Shows how to make an algorithm reusable

## Problem 3: Hard Problem - Generic Numeric Sum with Modern C#

This is a stronger interview example because older generic code could not safely use operators like `+` on `T` without workarounds. Modern C# supports this with generic math.

```csharp
using System.Numerics;

public static T Sum<T>(IEnumerable<T> items) where T : INumber<T>
{
    T total = T.Zero;

    foreach (var item in items)
    {
        total += item;
    }

    return total;
}
```

Usage:

```csharp
var intSum = Sum(new[] { 1, 2, 3, 4 });
var decimalSum = Sum(new[] { 1.5m, 2.5m, 3.0m });
```

Why this is a hard question:

- It tests whether you know that generic `T` cannot always use operators by default
- It tests modern C# knowledge
- It shows how constraints can describe capabilities, not just inheritance

## Problem 4: Hard Problem - Build a Generic Repository with Constraints

```csharp
public interface IEntity
{
    int Id { get; }
}

public class InMemoryRepository<T> where T : class, IEntity
{
    private readonly Dictionary<int, T> _store = new();

    public void Add(T entity)
    {
        _store[entity.Id] = entity;
    }

    public T? GetById(int id)
    {
        return _store.TryGetValue(id, out var entity) ? entity : null;
    }

    public IReadOnlyCollection<T> GetAll()
    {
        return _store.Values.ToList();
    }
}
```

What this shows in interview:

- Practical use of generic constraints
- Type-safe repository design
- One reusable implementation for any entity type

## Common Interview Mistakes

- Saying generics and `object` are basically the same
- Forgetting that constraints are needed when calling members on `T`
- Confusing variance with inheritance
- Thinking `List<string>` can be assigned to `List<object>`
- Forgetting the boxing/unboxing performance angle

## Quick Interview Summary

If asked for a short answer:

"Generics in C# let us write reusable, type-safe code using type parameters such as `T`. They are used in classes, methods, interfaces, and collections like `List<T>` and `Dictionary<TKey, TValue>`. They reduce casting, avoid boxing for many cases, and support constraints such as `where T : IComparable<T>` so algorithms can safely use required operations."

## References

- [Microsoft Learn - Generic types overview](https://learn.microsoft.com/en-us/dotnet/standard/generics)
- [Microsoft Learn - where generic type constraint](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint?redirectedfrom=MSDN)
- [Microsoft Learn - Generic Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-interfaces)
- [Microsoft Learn - Variance in Generic Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-generic-interfaces)
- [Microsoft Learn - Static virtual members in interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interface-implementation/static-virtual-interface-members)
