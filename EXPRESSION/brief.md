# Expression Brief for C# and .NET Core

## What Expression Means in C#

In interview context, `Expression` usually means **expression trees**, especially from:

```csharp
using System.Linq.Expressions;
```

An expression tree is a data structure that represents code as a tree of nodes.

Simple interview line:

"An expression tree represents code as data, so the program can inspect, modify, translate, or compile that code at runtime."

## Why Expression Trees Matter

Expression trees are important because frameworks can inspect your code instead of just executing it.

Common uses:

- Entity Framework translating LINQ into SQL
- dynamic query building
- rule engines
- filtering and sorting builders
- metadata-driven frameworks
- code generation style scenarios

## Basic Idea

This code:

```csharp
Expression<Func<int, int>> squareExpr = x => x * x;
```

does not directly store executable code only.

It stores a tree describing:

- parameter `x`
- operator `*`
- left side `x`
- right side `x`

That is why a framework can inspect it.

## `Expression` vs `Func`

This is one of the most common interview questions.

```csharp
Func<int, int> func = x => x * x;
Expression<Func<int, int>> expr = x => x * x;
```

Both look similar, but they are very different.

## Comparison Table: `Func` vs `Expression<Func<...>>`

| Topic | `Func<T, TResult>` | `Expression<Func<T, TResult>>` |
|---|---|---|
| What it stores | Executable delegate | Code represented as data |
| Can be inspected | No | Yes |
| Can be translated | No | Yes |
| Can be compiled and executed | Already executable | Yes, after calling `Compile()` |
| Common use | LINQ to Objects, callbacks, helpers | LINQ providers, EF Core, dynamic queries |
| Performance | Direct execution is faster | Has extra cost if compiled/visited |
| Best for | Running code | Understanding or translating code |

## How to Explain `Expression` with `Func`

Best interview answer:

"`Func` is executable code. `Expression<Func<...>>` is a data representation of code. If I pass `Func`, the receiver can only run it. If I pass `Expression<Func<...>>`, the receiver can inspect the body, read properties and operators, translate it to SQL, and optionally compile it later."

Example:

```csharp
Func<int, bool> f1 = x => x > 10;
Expression<Func<int, bool>> f2 = x => x > 10;
```

With `f1`:

- you can execute it
- you cannot easily inspect its internal logic

With `f2`:

- you can inspect `Body`, `Parameters`, `NodeType`
- you can translate it
- you can compile it later into a delegate

## Basic Example: Build and Execute an Expression

```csharp
using System;
using System.Linq.Expressions;

Expression<Func<int, int>> squareExpr = x => x * x;

Console.WriteLine(squareExpr); // x => (x * x)

Func<int, int> squareFunc = squareExpr.Compile();
Console.WriteLine(squareFunc(5)); // 25
```

Important point:

- expression tree = code as data
- `Compile()` turns it into executable code

## Problem Solving Example 1: Read Expression Details

This is a common interview-level example.

```csharp
using System;
using System.Linq.Expressions;

Expression<Func<int, bool>> expr = num => num < 5;

ParameterExpression parameter = expr.Parameters[0];
BinaryExpression body = (BinaryExpression)expr.Body;
ParameterExpression left = (ParameterExpression)body.Left;
ConstantExpression right = (ConstantExpression)body.Right;

Console.WriteLine($"Parameter: {parameter.Name}");
Console.WriteLine($"Operation: {body.NodeType}");
Console.WriteLine($"Right value: {right.Value}");
```

What this shows:

- how expression trees expose structure
- why frameworks can inspect conditions
- difference from normal delegates

## Problem Solving Example 2: Dynamic Filter Builder

This is a practical interview problem.

```csharp
using System;
using System.Linq.Expressions;

public static Expression<Func<Person, bool>> BuildAgeFilter(int minAge)
{
    ParameterExpression parameter = Expression.Parameter(typeof(Person), "p");
    MemberExpression property = Expression.Property(parameter, nameof(Person.Age));
    ConstantExpression constant = Expression.Constant(minAge);
    BinaryExpression body = Expression.GreaterThanOrEqual(property, constant);

    return Expression.Lambda<Func<Person, bool>>(body, parameter);
}

public class Person
{
    public string Name { get; set; } = "";
    public int Age { get; set; }
}
```

Usage:

```csharp
var filter = BuildAgeFilter(18);
Console.WriteLine(filter); // p => (p.Age >= 18)
```

Why this is good:

- shows manual expression construction
- very close to real query-builder code
- strong interview signal for backend roles

## Problem Solving Example 3: Safe Property Name Extraction

This is a very practical use of expressions.

```csharp
using System;
using System.Linq.Expressions;

public static string GetPropertyName<T, TProperty>(
    Expression<Func<T, TProperty>> expression)
{
    if (expression.Body is MemberExpression member)
        return member.Member.Name;

    throw new ArgumentException("Expression must target a property.");
}
```

Usage:

```csharp
string propertyName = GetPropertyName<Person, string>(p => p.Name);
Console.WriteLine(propertyName); // Name
```

Why this matters:

- avoids magic strings
- gives compile-time safety
- common in framework/helper code

## Hard Problem: Build a Dynamic Predicate Combiner

This is a strong interview-level expression tree problem.

```csharp
using System;
using System.Linq.Expressions;

public static class PredicateBuilder
{
    public static Expression<Func<T, bool>> And<T>(
        Expression<Func<T, bool>> left,
        Expression<Func<T, bool>> right)
    {
        ParameterExpression parameter = Expression.Parameter(typeof(T), "x");

        Expression leftBody = ReplaceParameter(left.Body, left.Parameters[0], parameter);
        Expression rightBody = ReplaceParameter(right.Body, right.Parameters[0], parameter);

        BinaryExpression body = Expression.AndAlso(leftBody, rightBody);

        return Expression.Lambda<Func<T, bool>>(body, parameter);
    }

    private static Expression ReplaceParameter(
        Expression body,
        ParameterExpression source,
        ParameterExpression target)
    {
        return new ParameterReplaceVisitor(source, target).Visit(body)!;
    }
}

public class ParameterReplaceVisitor : ExpressionVisitor
{
    private readonly ParameterExpression _source;
    private readonly ParameterExpression _target;

    public ParameterReplaceVisitor(ParameterExpression source, ParameterExpression target)
    {
        _source = source;
        _target = target;
    }

    protected override Expression VisitParameter(ParameterExpression node)
    {
        return node == _source ? _target : base.VisitParameter(node);
    }
}
```

Usage:

```csharp
Expression<Func<Person, bool>> isAdult = p => p.Age >= 18;
Expression<Func<Person, bool>> nameStartsWithA = p => p.Name.StartsWith("A");

var combined = PredicateBuilder.And(isAdult, nameStartsWithA);
Console.WriteLine(combined);
```

Why this is hard:

- requires parameter replacement
- requires `ExpressionVisitor`
- close to real-world dynamic query systems

## Hard Problem: Translate an Expression Tree

Another advanced interview topic is translating one expression into another.

Example idea:

- inspect constants
- replace constant `10` with `100`
- rebuild the tree

This is how frameworks transform logic before executing or translating it.

## Important Expression Types

- `Expression`
- `LambdaExpression`
- `ParameterExpression`
- `BinaryExpression`
- `MemberExpression`
- `ConstantExpression`
- `MethodCallExpression`
- `UnaryExpression`
- `ExpressionVisitor`

## Expression Tree Limitations

Important interview points:

- only expression lambdas can become expression trees
- statement lambdas cannot be converted directly
- not all newer C# constructs appear directly as new expression node types
- expression trees are immutable

This works:

```csharp
Expression<Func<int, int>> expr = x => x + 1;
```

This does not:

```csharp
// Expression<Func<int, int>> expr = x => { return x + 1; };
```

## Advantages

- code can be inspected
- code can be translated to another form
- supports dynamic query generation
- stronger than string-based query building
- useful for framework and library design

## Disadvantages

- more complex than normal delegates
- harder to debug
- usually slower to build/translate than direct code
- overkill for simple business logic

## When to Use `Func` and When to Use `Expression`

Use `Func` when:

- you only want to execute logic
- you need callbacks or computation
- you are working with in-memory collections

Use `Expression<Func<...>>` when:

- you need to inspect logic
- you need to translate logic to SQL or another target
- you are building dynamic queries or reusable framework logic

Interview shortcut:

- `Func` = run the code
- `Expression<Func<...>>` = understand the code

## Common Interview Mistakes

- saying expression trees are just delegates
- not understanding why EF Core needs expressions
- forgetting to mention `Compile()`
- ignoring that expression trees are immutable
- not knowing the difference between `IEnumerable` and `IQueryable` usage

## Quick Interview Summary

If asked for a short answer:

"An expression tree in C# represents code as a tree structure rather than just executable logic. `Func` lets you run code, but `Expression<Func<...>>` lets frameworks inspect and translate that code, which is why it is important in LINQ providers like Entity Framework."

## References

- [Microsoft Learn - Expression Trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/)
- [Microsoft Learn - Expression tree data structures](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-explained)
- [Microsoft Learn - Build expression trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-building)
- [Microsoft Learn - Execute expression trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-execution)
- [Microsoft Learn - Translate expression trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-translating)
