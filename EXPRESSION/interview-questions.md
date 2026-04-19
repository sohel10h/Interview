# C# Expression Tree Interview Questions and Answers

These are common expression-tree questions for C# and .NET interviews, grouped by difficulty and aligned with Microsoft documentation.

## Easy

1. What is an expression tree in C#?
   Answer: An expression tree is a data structure that represents code as a tree of expressions, so the code can be inspected, modified, translated, or compiled at runtime.
2. Which namespace is used for expression trees?
   Answer: `System.Linq.Expressions`.
3. What is the difference between `Func<T, TResult>` and `Expression<Func<T, TResult>>`?
   Answer: `Func` is executable code. `Expression<Func<...>>` stores the code as data that can be inspected and translated.
4. Why are expression trees important?
   Answer: They let frameworks inspect your code structure, which is essential for dynamic queries and LINQ providers such as Entity Framework.
5. Can expression trees be executed?
   Answer: Yes. They can be compiled using `Compile()` into a delegate and then executed.
6. What does `Compile()` do?
   Answer: It converts an expression tree into an executable delegate.
7. What kind of lambda can become an expression tree?
   Answer: Expression lambdas, such as `x => x + 1`.
8. Can statement-bodied lambdas become expression trees?
   Answer: No, not directly.
9. What is `ParameterExpression`?
   Answer: It represents a parameter node in an expression tree.
10. What is `BinaryExpression`?
    Answer: It represents a binary operation such as `+`, `-`, `==`, or `>`.

## Medium

1. Why does Entity Framework use `Expression<Func<T, bool>>` instead of `Func<T, bool>`?
   Answer: Because EF needs to inspect the logic and translate it into SQL. A plain `Func` can only be executed in memory.
2. What is the difference between `IEnumerable` and `IQueryable` in this discussion?
   Answer: `IEnumerable` usually works with in-memory delegates, while `IQueryable` carries an expression tree so a provider can translate the query.
3. What is `ExpressionVisitor` used for?
   Answer: It is used to traverse and transform expression trees.
4. Are expression trees mutable?
   Answer: No. They are immutable, so modifications require building a new tree.
5. How do you inspect a property access in an expression tree?
   Answer: Usually by examining a `MemberExpression`.
6. What is a common practical use of expression trees apart from EF?
   Answer: Dynamic filtering, sorting builders, validation helpers, rule engines, and property-name extraction without magic strings.
7. What happens if you pass a `Func<T, bool>` to a query provider?
   Answer: The provider cannot inspect or translate the logic, so it usually runs in memory after data is retrieved.
8. How do you manually create an expression tree?
   Answer: Use factory methods such as `Expression.Parameter`, `Expression.Constant`, `Expression.Property`, and `Expression.Lambda`.
9. Why are expression trees better than string-based dynamic queries?
   Answer: Because they are strongly typed, safer to refactor, and checked by the compiler.
10. What is `MethodCallExpression`?
    Answer: It represents a method call inside an expression tree.

## Hard

1. How would you explain expression trees in one line?
   Answer: Expression trees represent code as data so .NET can understand and transform logic before executing it.
2. Why is `Expression<Func<T, bool>>` more powerful than `Func<T, bool>`?
   Answer: Because it preserves the structure of the code, including operators, property access, and method calls, which makes translation and analysis possible.
3. How do you combine two predicates dynamically?
   Answer: Replace their parameters so both use the same parameter, then combine the bodies with `Expression.AndAlso` or `Expression.OrElse`.
4. What role does parameter replacement play in predicate composition?
   Answer: It ensures both expression bodies refer to the same logical parameter, which is required when building a valid combined tree.
5. Why are expression trees important for building query providers?
   Answer: Because query providers inspect the expression tree and translate it into another language or execution model, such as SQL.
6. What are some limitations of expression trees?
   Answer: They support only expression lambdas, are immutable, and do not directly represent every modern C# language feature as unique node types.
7. What is translation of an expression tree?
   Answer: It is the process of visiting the tree and building a new tree or another representation, such as SQL or a modified expression.
8. What is the downside of overusing expression trees?
   Answer: Code becomes more complex, harder to debug, and unnecessary for simple logic that only needs direct execution.
9. When should you prefer `Func` over an expression tree?
   Answer: When the logic only needs to be executed and there is no need to inspect or translate it.
10. When should you prefer an expression tree over `Func`?
    Answer: When the receiver must understand the code structure, such as for LINQ providers, dynamic query builders, or framework utilities.

## Very Common `Expression` vs `Func` Questions

1. What is the main difference between `Func` and `Expression<Func<...>>`?
   Answer: `Func` stores executable logic. `Expression<Func<...>>` stores a tree describing that logic.
2. Why can EF Core translate `Expression<Func<User, bool>>` but not a normal `Func<User, bool>`?
   Answer: Because the expression tree exposes the code structure, while the normal delegate hides it.
3. Can an expression tree be executed like `Func`?
   Answer: Yes, but first it must be compiled using `Compile()`.
4. Which one should you use for in-memory filtering?
   Answer: Usually `Func<T, bool>`.
5. Which one should you use for provider-translated queries?
   Answer: Usually `Expression<Func<T, bool>>`.

## Very Common Coding Questions

1. Write an expression tree for squaring a number and execute it.
   Answer:

```csharp
Expression<Func<int, int>> expr = x => x * x;
Func<int, int> compiled = expr.Compile();
int result = compiled(5);
```

2. Write code to inspect a simple expression tree.
   Answer:

```csharp
Expression<Func<int, bool>> expr = x => x > 10;
var parameter = expr.Parameters[0];
var body = (BinaryExpression)expr.Body;
```

3. Write code to build an expression tree manually.
   Answer:

```csharp
ParameterExpression p = Expression.Parameter(typeof(int), "x");
BinaryExpression body = Expression.Multiply(p, p);
Expression<Func<int, int>> expr =
    Expression.Lambda<Func<int, int>>(body, p);
```

4. Write code to extract a property name safely.
   Answer:

```csharp
public static string GetPropertyName<T, TProperty>(
    Expression<Func<T, TProperty>> expression)
{
    return ((MemberExpression)expression.Body).Member.Name;
}
```

## .NET Core Follow-Up Questions

1. Where are expression trees commonly used in .NET Core projects?
   Answer: In Entity Framework, dynamic filtering, query composition, metadata-driven helpers, and reusable framework utilities.
2. Why do ORMs prefer expression trees for queries?
   Answer: Because they can inspect the query logic and translate it into SQL rather than executing it immediately in memory.
3. Why is expression-tree immutability important?
   Answer: It makes the structure stable and predictable, but it also means modifications require rebuilding parts of the tree.
4. Can expression trees be used with delegates together?
   Answer: Yes. An expression tree can be compiled into a delegate when execution is needed.
5. What is the shortest way to explain `Expression` with `Func` in interview?
   Answer: `Func` runs code. `Expression<Func<...>>` describes code.

## Recommended Answer Style

For interview answers:

- start with "code as data"
- compare it directly with `Func`
- mention EF Core or LINQ provider translation
- mention `Compile()`
- for advanced rounds, mention `ExpressionVisitor` and immutable trees

## Sources

Recurring interview-question patterns were taken from:

- [Stack Overflow - Expression Tree](https://stackoverflow.com/questions/2533649/expression-tree)
- [Stack Overflow - Practical use of expression trees](https://stackoverflow.com/questions/403088/practical-use-of-expression-trees)

Technical alignment was checked against:

- [Microsoft Learn - Expression Trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/)
- [Microsoft Learn - Expression tree data structures](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-explained)
- [Microsoft Learn - Build expression trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-building)
- [Microsoft Learn - Execute expression trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-execution)
- [Microsoft Learn - Interpret expressions](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-interpreting)
- [Microsoft Learn - Translate expression trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/expression-trees-translating)
