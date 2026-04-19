# C# Func Interview Questions and Answers

These are common `Func` questions for C# and .NET interviews, grouped by difficulty and aligned with Microsoft documentation.

## Easy

1. What is `Func` in C#?
   Answer: `Func` is a built-in generic delegate that represents a method returning a value.
2. Is `Func` a delegate?
   Answer: Yes. `Func` is a predefined generic delegate type provided by .NET.
3. What is the difference between `Func` and `Action`?
   Answer: `Func` returns a value. `Action` does not return a value.
4. What is the difference between `Func<T, bool>` and `Predicate<T>`?
   Answer: Both can represent logic returning `bool` for one input, but `Predicate<T>` is a specialized delegate intended specifically for boolean testing.
5. In `Func<int, string>`, what do the types mean?
   Answer: It takes an `int` as input and returns a `string`.
6. In `Func<int, int, int>`, which type is the return type?
   Answer: The last type argument is the return type.
7. Can a lambda expression be assigned to a `Func`?
   Answer: Yes, if the lambda signature matches the `Func` signature.
8. When should you use `Func`?
   Answer: Use it when you need to pass a method or lambda that returns a value and the callback is general-purpose.
9. Can `Func` have zero input parameters?
   Answer: Yes. Example: `Func<int>` returns an `int` and takes no input.
10. Why is `Func` used so often in modern C#?
    Answer: Because it works naturally with lambdas, LINQ, callbacks, transformations, and helper methods.

## Medium

1. What is the main advantage of `Func` over a custom delegate?
   Answer: It reduces boilerplate and is concise for common callback scenarios.
2. When is a custom delegate better than `Func`?
   Answer: When a named delegate improves readability or expresses domain meaning better.
3. Can `Func` be used as a method parameter?
   Answer: Yes, and that is one of its most common uses.
4. Can `Func` be returned from a method?
   Answer: Yes. Methods can return `Func` values, which is useful for composition and factory-like behavior.
5. How is `Func` used in LINQ?
   Answer: LINQ methods such as `Select` and `Where` accept lambdas that correspond to delegate signatures like `Func<T, TResult>` or `Func<T, bool>`.
6. What is deferred execution in relation to `Func`?
   Answer: It means the work described by the `Func` is not executed until the delegate is actually invoked.
7. Can `Func` capture local variables?
   Answer: Yes. Lambdas assigned to `Func` can capture variables from their surrounding scope.
8. What is a closure in this context?
   Answer: A closure is created when a lambda stored in a `Func` captures outer local variables.
9. Is `Func` type-safe?
   Answer: Yes. Only methods or lambdas with compatible input and return types can be assigned.
10. Can `Func` replace all custom delegates?
   Answer: No. It is useful for common cases, but not every API is clearer with `Func`.

## Hard

1. How would you explain `Func` to an interviewer in one line?
   Answer: `Func` is a built-in generic delegate for representing reusable behavior that returns a value.
2. Why is `Func` useful for strategy-style design?
   Answer: Because you can pass different implementations of the same calculation or transformation without changing the calling code.
3. How can `Func` help build reusable wrappers like logging, retry, or caching?
   Answer: By accepting an operation as `Func<T>`, the wrapper can execute that operation while adding extra behavior around it.
4. What is the tradeoff of overusing `Func`?
   Answer: Overuse can make code less expressive than a well-named custom delegate or interface.
5. What is function composition with `Func`?
   Answer: It is combining multiple functions so the output of one becomes the input of the next.
6. When would you choose `Func<T, bool>` over `Predicate<T>`?
   Answer: `Func<T, bool>` is often preferred in APIs and LINQ-heavy code because it matches the broader `Func` family and composes consistently with other delegate forms.
7. Can `Func` be part of a pipeline design?
   Answer: Yes. Multiple `Func` values can be chained together to transform data step by step.
8. What are common performance concerns with `Func` and lambdas?
   Answer: Captured closures and unnecessary allocations can add overhead in performance-critical code.
9. Can `Func` be asynchronous?
   Answer: A `Func` can return `Task` or `Task<T>`, for example `Func<Task<int>>`, but `Func` itself is not automatically asynchronous.
10. When should an interface be preferred over `Func`?
    Answer: When the behavior is larger than a single callback or needs multiple methods, configuration, or state.

## Very Common Coding Questions

1. Write a method that accepts a `Func<int, int, int>` and executes it.
   Answer:

```csharp
public static int Execute(int a, int b, Func<int, int, int> operation)
{
    return operation(a, b);
}
```

2. Write a generic mapper using `Func`.
   Answer:

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

3. Write a retry helper using `Func<T>`.
   Answer:

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

4. Write a function composer using `Func`.
   Answer:

```csharp
public static Func<TInput, TOutput> Compose<TInput, TMiddle, TOutput>(
    Func<TInput, TMiddle> first,
    Func<TMiddle, TOutput> second)
{
    return input => second(first(input));
}
```

## `Func` vs Delegate Quick Comparison

1. Is `Func` different from delegate?
   Answer: `Func` is not a separate concept. It is one of the predefined delegate types in .NET.
2. Why do many developers prefer `Func`?
   Answer: Because it is concise and works very well with lambdas and generic APIs.
3. Why might a custom delegate still be better?
   Answer: Because a named delegate can describe intent more clearly than a generic `Func<...>` signature.

## .NET Core Follow-Up Questions

1. Where is `Func` commonly used in .NET Core projects?
   Answer: In LINQ, DI factories, middleware helpers, caching wrappers, retry logic, mapping functions, and configuration callbacks.
2. Why is `Func<T>` useful for lazy or delayed execution?
   Answer: Because it stores work that can be executed later only when needed.
3. Can dependency injection use `Func`?
   Answer: Yes. `Func<T>` can be used in some factory-style scenarios to defer object creation or resolve dependencies later.
4. Why is `Func` important for middleware and pipeline design?
   Answer: Because each step can be modeled as behavior that transforms or computes a result and passes it onward.
5. Is `Func` enough for all extensibility scenarios?
   Answer: No. For larger contracts or richer behaviors, interfaces or named delegates are often better.

## Recommended Answer Style

For interview answers:

- start with "built-in generic delegate"
- say that it returns a value
- compare it briefly with `Action`
- mention when to use `Func` instead of a custom delegate
- give one LINQ or callback example

## Sources

Recurring interview-question patterns were taken from:

- [CodingInterview.com - Delegates, Events, and Lambdas](https://www.codinginterview.com/guide/c-sharp-coding-interview-questions/)
- [Microsoft Q&A - Func, Action, Predicate delegates, events in C#](https://learn.microsoft.com/en-us/answers/questions/1664718/func-action-predicate-delegates-events-in-c)
- [Stack Overflow - Func vs. Action vs. Predicate](https://stackoverflow.com/questions/8511066/func-vs-action-vs-predicate)

Technical alignment was checked against:

- [Microsoft Learn - Func<TResult> Delegate](https://learn.microsoft.com/en-us/dotnet/api/system.func-1?view=net-10.0)
- [Microsoft Learn - Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [Microsoft Learn - Generic Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-delegates)
- [Microsoft Learn - Lambda expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions)
