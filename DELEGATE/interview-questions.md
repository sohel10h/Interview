# C# Delegate Interview Questions and Answers

These are common delegate questions for C# and .NET interviews, grouped by difficulty and aligned with Microsoft documentation.

## Easy

1. What is a delegate in C#?
   Answer: A delegate is a type-safe reference to a method. It can store and invoke any method that matches its signature.
2. Why do we use delegates?
   Answer: Delegates are used to pass behavior as parameters, support callbacks, implement events, and enable flexible reusable logic.
3. What does type-safe mean for delegates?
   Answer: It means only methods with a compatible parameter list and return type can be assigned to a delegate.
4. How do delegates differ from normal method calls?
   Answer: A normal method call is fixed in code, while a delegate lets the target method be selected and invoked dynamically.
5. What is the basic syntax to declare a delegate?
   Answer:

```csharp
public delegate int Operation(int x, int y);
```

6. What are built-in generic delegates in C#?
   Answer: `Action`, `Func`, and `Predicate<T>`.
7. What is `Action`?
   Answer: `Action` is a delegate type for methods that return `void`.
8. What is `Func`?
   Answer: `Func` is a delegate type for methods that return a value. The last type parameter is the return type.
9. What is `Predicate<T>`?
   Answer: It is a delegate that takes one parameter of type `T` and returns `bool`.
10. Can a delegate point to both static and instance methods?
    Answer: Yes, as long as the method signature matches the delegate type.

## Medium

1. What is a multicast delegate?
   Answer: A multicast delegate contains more than one method in its invocation list and can call them in order.
2. How do you combine delegates?
   Answer: Use `+` or `+=` to combine them, and `-` or `-=` to remove handlers.
3. What happens if one method in a multicast delegate throws an exception?
   Answer: The invocation normally stops and the exception propagates unless you invoke the handlers individually with error handling.
4. What is the difference between a delegate and an event?
   Answer: A delegate is a method reference type. An event uses a delegate internally but restricts who can invoke it, usually only the owning class.
5. Can delegates return values?
   Answer: Yes. A delegate can return a value if its signature defines one.
6. What is the issue with return values in multicast delegates?
   Answer: When a multicast delegate has a return type, only the last invoked method's return value is returned.
7. What is the relation between delegates and lambda expressions?
   Answer: Lambda expressions can be assigned to delegate types when their signatures match.
8. What is an anonymous method?
   Answer: An anonymous method is an inline unnamed method created using the `delegate` keyword. Lambdas are the more common modern syntax.
9. Can delegates be passed as method parameters?
   Answer: Yes, and that is one of their main uses.
10. Why are delegates important in LINQ and callbacks?
    Answer: Because LINQ operators and callback-based APIs frequently accept behavior as functions, usually through delegates.

## Hard

1. How do delegates relate to events in .NET?
   Answer: Events are built on delegates. A delegate defines the handler signature, while the event provides controlled subscription and invocation behavior.
2. What is an invocation list?
   Answer: It is the list of methods stored inside a multicast delegate.
3. How can you inspect each method in a multicast delegate?
   Answer: Use `GetInvocationList()`.
4. What is method group conversion?
   Answer: It is the compiler feature that allows assigning a method name directly to a delegate without explicitly calling the constructor.
5. Can delegates capture local variables?
   Answer: Yes, when used with lambdas or anonymous methods, they can close over local variables.
6. What is a closure in delegate-related discussion?
   Answer: A closure is created when a lambda or anonymous method captures variables from its outer scope.
7. What are the performance considerations of delegates?
   Answer: Delegates are generally efficient, but excessive allocations, captured closures, and repeated dynamic combinations can add overhead.
8. When should you prefer interfaces over delegates?
   Answer: Prefer interfaces when you need a richer contract with multiple operations or state. Prefer delegates for a single behavior callback.
9. How would you safely invoke a multicast delegate when one handler might fail?
   Answer: Iterate through `GetInvocationList()` and invoke each handler in a separate `try/catch`.
10. What is the difference between custom delegates and `Func`/`Action`?
    Answer: `Func` and `Action` are built-in generic delegate types used for common cases. Custom delegates are useful when a named delegate improves readability or domain meaning.

## Very Common Coding Questions

1. Write code to pass a method as a parameter using a delegate.
   Answer:

```csharp
public delegate int Operation(int x, int y);

public static int Execute(int a, int b, Operation operation)
{
    return operation(a, b);
}
```

2. Write code using `Func` to add two numbers.
   Answer:

```csharp
Func<int, int, int> add = (x, y) => x + y;
int result = add(10, 20);
```

3. Write code for a multicast delegate.
   Answer:

```csharp
Action handler = () => Console.WriteLine("First");
handler += () => Console.WriteLine("Second");
handler();
```

4. Write code to invoke each multicast handler safely.
   Answer:

```csharp
public static void SafeInvoke(Action handlers)
{
    foreach (Delegate singleHandler in handlers.GetInvocationList())
    {
        try
        {
            ((Action)singleHandler)();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
    }
}
```

## .NET Core Follow-Up Questions

1. Where are delegates heavily used in .NET or ASP.NET Core?
   Answer: Delegates are used in events, LINQ, middleware pipelines, task continuations, callbacks, and many framework extension points.
2. Why are delegates important for middleware-like design?
   Answer: Because each step can receive behavior and pass control to the next step, which is similar to a processing pipeline.
3. Why do modern C# developers use `Func` and `Action` so often?
   Answer: They reduce boilerplate and work naturally with lambdas and APIs that accept behavior.
4. Are events just delegates?
   Answer: Not exactly. Events are built on delegates, but they add encapsulation around subscription and invocation.
5. When would you define a custom delegate in a real project?
   Answer: When a named delegate makes the intent clearer than `Func` or `Action`, especially in domain-specific APIs.

## Recommended Answer Style

For interview answers:

- Start with "type-safe method reference"
- mention callbacks, events, and lambdas
- explain `Action`, `Func`, and `Predicate`
- for experienced roles, mention multicast delegates and closures

## Sources

Recurring interview-question patterns were taken from:

- [Dot Net Tutorials - Delegate Interview Questions and Answers in C#](https://dotnettutorials.net/lesson/delegate-interview-questions-answers-csharp/)
- [ByteHide - C# Delegates Interview Questions](https://www.bytehide.com/blog/csharp-delegates-interview-questions)
- [IndiaBIX - Delegates Questions and Answers](https://www.indiabix.com/c-sharp-programming/delegates/)

Technical alignment was checked against:

- [Microsoft Learn - Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [Microsoft Learn - System.Delegate and the delegate keyword](https://learn.microsoft.com/en-us/dotnet/csharp/delegate-class)
- [C# Language Specification - Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/delegates)
