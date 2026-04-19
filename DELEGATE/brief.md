# Delegate Brief for C# and .NET Core

## What a Delegate Is

A delegate in C# is a type that stores a reference to a method with a specific parameter list and return type.

Simple interview line:

"A delegate is a type-safe method reference. It lets us pass methods as arguments and invoke them later."

## Why Delegates Matter

Delegates are used for:

- callbacks
- event handling
- strategy-style behavior
- LINQ and lambda expressions
- middleware-style pipelines
- flexible method passing

They are a core part of modern C# because lambdas, `Action`, `Func`, `Predicate`, and events all build on delegate concepts.

## Basic Syntax

```csharp
public delegate int Operation(int x, int y);
```

This delegate can point to any method that:

- takes two `int` parameters
- returns an `int`

Example:

```csharp
public static int Add(int a, int b) => a + b;
public static int Multiply(int a, int b) => a * b;

Operation op = Add;
Console.WriteLine(op(2, 3)); // 5

op = Multiply;
Console.WriteLine(op(2, 3)); // 6
```

## How Delegates Work

When a method matches the delegate signature, C# can store that method inside a delegate instance.

Then you can:

- pass it to another method
- store it in a variable
- invoke it later
- combine multiple methods in some cases

## Problem Solving Example 1: Pass Behavior as a Parameter

This is one of the best ways to explain delegates in an interview.

```csharp
public delegate int Operation(int x, int y);

public static int Execute(int a, int b, Operation operation)
{
    return operation(a, b);
}

public static int Add(int x, int y) => x + y;
public static int Subtract(int x, int y) => x - y;

int sum = Execute(10, 5, Add);
int difference = Execute(10, 5, Subtract);
```

What this shows:

- behavior can be injected
- logic stays reusable
- delegate acts like a strategy

## Problem Solving Example 2: Use Built-in Generic Delegates

In real projects, custom delegates are often replaced by built-in delegate types.

```csharp
Func<int, int, int> add = (a, b) => a + b;
Action<string> print = message => Console.WriteLine(message);
Predicate<int> isEven = number => number % 2 == 0;

Console.WriteLine(add(2, 3));   // 5
print("Hello");
Console.WriteLine(isEven(10));  // True
```

Remember:

- `Action`: no return value
- `Func`: returns a value
- `Predicate<T>`: returns `bool`

## Problem Solving Example 3: Multicast Delegate

Delegates can hold more than one method in the invocation list.

```csharp
public delegate void Notifier(string message);

public static void SendEmail(string message) =>
    Console.WriteLine($"Email: {message}");

public static void SendSms(string message) =>
    Console.WriteLine($"SMS: {message}");

Notifier notifier = SendEmail;
notifier += SendSms;

notifier("Order created");
```

Output:

- `Email: Order created`
- `SMS: Order created`

Interview point:

Multicast delegates are common in event-style programming.

## Delegates and Events

Events are built on delegates.

Example:

```csharp
public class Publisher
{
    public event Action? DataChanged;

    public void Raise()
    {
        DataChanged?.Invoke();
    }
}
```

Important distinction:

- delegate: can be invoked directly if you have access
- event: only the owning class can raise it

## Anonymous Methods and Lambda Expressions

Delegates work very naturally with lambdas.

```csharp
Func<int, int, int> max = (a, b) => a > b ? a : b;
Console.WriteLine(max(10, 20));
```

Lambdas are usually the most common delegate usage in modern C#.

## Hard Problem: Build a Simple Processing Pipeline

This is a stronger interview-level example because it shows delegates used as reusable behavior blocks.

```csharp
using System;
using System.Collections.Generic;

public delegate string TextStep(string input);

public static class TextPipeline
{
    public static string Run(string input, IEnumerable<TextStep> steps)
    {
        string result = input;

        foreach (var step in steps)
        {
            result = step(result);
        }

        return result;
    }
}
```

Usage:

```csharp
var steps = new List<TextStep>
{
    text => text.Trim(),
    text => text.ToUpper(),
    text => $"[{text}]"
};

string output = TextPipeline.Run("  hello delegate  ", steps);
Console.WriteLine(output); // [HELLO DELEGATE]
```

Why this is hard:

- it uses delegates for composable behavior
- it models a pipeline pattern
- it is close to how middleware and processing chains work

## Hard Problem: Safe Invocation of Multicast Delegates

This is a common advanced interview discussion point.

If one method in a multicast delegate throws an exception, later methods are normally not executed.

One way to handle this is to invoke each handler separately:

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
            Console.WriteLine($"Handler failed: {ex.Message}");
        }
    }
}
```

Why this matters:

- shows knowledge of invocation lists
- shows multicast delegate behavior
- shows defensive coding

## Advantages

- flexible design
- type-safe callbacks
- cleaner separation of behavior
- strong support for functional-style patterns
- essential for events and LINQ

## Limitations

- can make debugging harder if overused
- multicast delegates with return values are tricky
- exceptions in invocation chains must be handled carefully
- direct method calls are simpler when behavior does not need to vary

## Common Interview Mistakes

- saying delegate is exactly the same as event
- forgetting that method signature must match
- not knowing `Action`, `Func`, and `Predicate`
- not understanding multicast behavior
- confusing delegate with anonymous method or lambda itself

## Quick Interview Summary

If asked for a short answer:

"A delegate in C# is a type-safe reference to a method. It allows methods to be passed as parameters, stored, and invoked later. Delegates are widely used in callbacks, events, and lambda-based APIs. In modern C#, built-in delegates like `Action`, `Func`, and `Predicate` are used very often."

## References

- [Microsoft Learn - Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [Microsoft Learn - System.Delegate and the delegate keyword](https://learn.microsoft.com/en-us/dotnet/csharp/delegate-class)
- [C# Language Specification - Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/delegates)
