# C# Generics Interview Questions and Answers

These are common C# generics interview questions collected from recurring interview-prep sources and aligned with Microsoft documentation.

## Easy

1. What are generics in C#?
   Answer: Generics allow you to define classes, methods, interfaces, and delegates with placeholders for data types. This makes code reusable and type-safe.
2. Why do we use generics?
   Answer: We use generics for code reuse, compile-time type safety, better readability, and better performance than `object`-based code in many cases.
3. What is the difference between generics and `object`?
   Answer: Generics preserve the actual type and avoid many casts. `object` can store anything, but it loses type information and may require casting and boxing.
4. Give examples of built-in generic types in .NET.
   Answer: Common examples are `List<T>`, `Dictionary<TKey, TValue>`, `Queue<T>`, `Stack<T>`, `IEnumerable<T>`, and `Nullable<T>`.
5. What is a generic method?
   Answer: A generic method defines its own type parameter, such as `T`, and can work with multiple data types without duplicating code.
6. What is a generic class?
   Answer: A generic class uses one or more type parameters so the same class can work with different data types.
7. What is a type parameter?
   Answer: A type parameter is a placeholder type like `T`, `TKey`, or `TValue` used in a generic type or method.
8. What problem do generics solve in collections?
   Answer: They prevent mixed-type storage, reduce runtime casting errors, and improve performance for value types.
9. Can we create generic interfaces in C#?
   Answer: Yes. For example, `IEnumerable<T>` and `IComparer<T>` are generic interfaces.
10. Can generic code improve performance?
    Answer: Yes. It can reduce boxing and unboxing for value types and remove many runtime casts.

## Medium

1. What are generic constraints in C#?
   Answer: Constraints restrict which types can be used with a generic parameter. They tell the compiler what the type must support.
2. Why do we use `where T : IComparable<T>`?
   Answer: It ensures that `T` supports comparison, so code can safely call `CompareTo`.
3. What does `where T : class` mean?
   Answer: It restricts `T` to reference types.
4. What does `where T : struct` mean?
   Answer: It restricts `T` to non-nullable value types.
5. What does `where T : new()` mean?
   Answer: It requires `T` to have a public parameterless constructor.
6. Can a generic class have multiple type parameters?
   Answer: Yes. For example, `Dictionary<TKey, TValue>` has two type parameters.
7. What is the difference between an open generic type and a closed generic type?
   Answer: An open generic type still has type parameters, like `List<T>`. A closed generic type has actual types supplied, like `List<int>`.
8. What is boxing, and how do generics help?
   Answer: Boxing converts a value type to `object` or an interface type. Generics help avoid this in many scenarios by preserving the actual value type.
9. Can constructors be generic in C#?
   Answer: No. Classes can be generic, but constructors themselves cannot declare their own type parameters.
10. Can methods inside a generic class have their own type parameters?
    Answer: Yes. A method can define its own generic type parameters even if the class is already generic.

## Hard

1. What is covariance in generics?
   Answer: Covariance allows a more derived type to be used where a less derived type is expected. In C#, it is supported in some interfaces and delegates using `out`.
2. What is contravariance in generics?
   Answer: Contravariance allows a less derived type to be used where a more derived type is expected. In C#, it is supported in some interfaces and delegates using `in`.
3. Why is `IEnumerable<string>` assignable to `IEnumerable<object>`, but `List<string>` is not assignable to `List<object>`?
   Answer: Because `IEnumerable<T>` is covariant, but `List<T>` is invariant.
4. Do value types support variance?
   Answer: No. Variance applies to reference types.
5. Why can't you directly use operators like `+` on generic `T` in older C# generic code?
   Answer: Because the compiler does not know that every possible `T` supports that operator. You need suitable constraints or modern generic math support.
6. What is generic math in modern C#?
   Answer: Generic math uses interfaces with static abstract members, such as `INumber<T>`, so generic algorithms can use numeric operators safely.
7. When would you use generic interfaces over non-generic interfaces?
   Answer: When you want strong typing, better discoverability, and reduced casting or boxing.
8. How would you design a reusable repository using generics?
   Answer: Create a generic repository type such as `Repository<T>` and use constraints like `where T : IEntity` if the code needs members such as `Id`.
9. What is the risk of overusing generics?
   Answer: Code can become too abstract, harder to read, and harder to debug if type parameters and constraints are not meaningful.
10. What is the difference between C# generics and C++ templates?
    Answer: C# generics are runtime-supported by the CLR and are more constrained and type-safe. C++ templates are more flexible and support specialization more deeply.

## Very Common Coding Questions

1. Write a generic method to swap two values.
   Answer:

```csharp
public static void Swap<T>(ref T left, ref T right)
{
    T temp = left;
    left = right;
    right = temp;
}
```

2. Write a generic method to return the maximum of two values.
   Answer:

```csharp
public static T Max<T>(T a, T b) where T : IComparable<T>
{
    return a.CompareTo(b) >= 0 ? a : b;
}
```

3. Write a generic method to search for an item in a list.
   Answer:

```csharp
public static bool ContainsItem<T>(IEnumerable<T> items, T target)
{
    foreach (var item in items)
    {
        if (EqualityComparer<T>.Default.Equals(item, target))
            return true;
    }

    return false;
}
```

4. Write a generic sum method for numeric types in modern C#.
   Answer:

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

## .NET Core Follow-Up Questions

1. Why are generic collections preferred over old non-generic collections in .NET Core?
   Answer: Because they are type-safe, clearer, and often faster due to reduced boxing and casting.
2. Where do you use generics most in ASP.NET Core projects?
   Answer: In collections, repositories, service abstractions, result wrappers, middleware helpers, logging helpers, and DTO or mapping utilities.
3. Can dependency injection work with open generic registrations?
   Answer: Yes. ASP.NET Core DI supports open generic registrations such as mapping `IRepository<>` to `Repository<>`.
4. What is an open generic registration example in DI?
   Answer:

```csharp
services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
```

5. Why are generics useful in repository or service layers?
   Answer: They reduce duplicated CRUD-style code and enforce consistent behavior across entity types.

## Recommended Answer Style

For interview answers:

- Start with what generics solve
- Mention type safety and code reuse
- Add one practical .NET example like `List<T>` or `Dictionary<TKey, TValue>`
- For experienced interviews, mention constraints or variance

## Sources

Recurring interview-question patterns were taken from:

- [Naukri Code 360 - C# Generics Interview Questions](https://www.naukri.com/code360/library/csharp-generics-interview-questions)
- [Interview Coder - C# Interview Questions](https://www.interviewcoder.co/blog/c-hash-interview-questions)

Technical alignment was checked against:

- [Microsoft Learn - Generic types overview](https://learn.microsoft.com/en-us/dotnet/standard/generics)
- [Microsoft Learn - where generic type constraint](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint?redirectedfrom=MSDN)
- [Microsoft Learn - Generic Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-interfaces)
- [Microsoft Learn - Variance in Generic Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-generic-interfaces)
- [Microsoft Learn - Static virtual members in interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interface-implementation/static-virtual-interface-members)
