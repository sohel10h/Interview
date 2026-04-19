# Reflection Brief for C# and .NET Core

## What Reflection Is

Reflection in C# means inspecting metadata about assemblies, types, methods, properties, fields, attributes, and constructors at runtime.

It lets the program discover information about code even if the exact type was not known at compile time.

Core namespace:

```csharp
using System.Reflection;
```

Interview line:

"Reflection lets .NET inspect metadata and interact with types dynamically at runtime."

## Why Reflection Matters

Common real uses:

- Dependency injection containers
- ORMs
- Serializers
- Model binders
- Plugin systems
- Attribute-based validation
- Test frameworks

Reflection is powerful, but it is slower and less safe than direct strongly typed code.

## Common Reflection Types

- `Type`
- `Assembly`
- `PropertyInfo`
- `MethodInfo`
- `FieldInfo`
- `ConstructorInfo`
- `ParameterInfo`
- `Attribute`

## Basic Example: Read Type Information

```csharp
using System;
using System.Reflection;

public class Person
{
    public string Name { get; set; } = "";
    public int Age { get; set; }

    public void SayHello()
    {
        Console.WriteLine($"Hello, I am {Name}");
    }
}

Type type = typeof(Person);

Console.WriteLine(type.Name);

foreach (PropertyInfo property in type.GetProperties())
{
    Console.WriteLine($"{property.Name} : {property.PropertyType.Name}");
}

foreach (MethodInfo method in type.GetMethods())
{
    Console.WriteLine(method.Name);
}
```

## Problem Solving Example 1: Create Object Dynamically

This is a common interview problem.

```csharp
using System;

public class Calculator
{
    public int Add(int a, int b) => a + b;
}

Type type = typeof(Calculator);
object? instance = Activator.CreateInstance(type);

if (instance is not null)
{
    MethodInfo? method = type.GetMethod("Add");
    object? result = method?.Invoke(instance, new object[] { 10, 20 });
    Console.WriteLine(result); // 30
}
```

What this shows:

- runtime type discovery
- object creation without `new Calculator()`
- dynamic method invocation

## Problem Solving Example 2: Read Property Values from Any Object

```csharp
using System;
using System.Reflection;

public static void PrintProperties<T>(T obj)
{
    Type type = typeof(T);

    foreach (PropertyInfo property in type.GetProperties())
    {
        object? value = property.GetValue(obj);
        Console.WriteLine($"{property.Name} = {value}");
    }
}
```

Usage:

```csharp
var person = new Person { Name = "Sohel", Age = 25 };
PrintProperties(person);
```

Why interviewers like this:

- simple reflection usage
- reusable helper
- shows `PropertyInfo` and `GetValue`

## Problem Solving Example 3: Attribute-Based Validation

This is a more practical example.

```csharp
using System;
using System.Linq;
using System.Reflection;

[AttributeUsage(AttributeTargets.Property)]
public class RequiredAttribute : Attribute
{
}

public class UserInput
{
    [Required]
    public string Email { get; set; } = "";

    public string? Phone { get; set; }
}

public static class Validator
{
    public static bool IsValid<T>(T obj)
    {
        Type type = typeof(T);

        foreach (PropertyInfo property in type.GetProperties())
        {
            bool isRequired = property.GetCustomAttributes(typeof(RequiredAttribute), true).Any();

            if (!isRequired)
                continue;

            object? value = property.GetValue(obj);

            if (value is null || string.IsNullOrWhiteSpace(value.ToString()))
                return false;
        }

        return true;
    }
}
```

Why this is good:

- shows reflection + attributes together
- looks like real framework code
- demonstrates metadata-driven behavior

## Hard Problem: Mini Plugin Loader

This is a strong interview-level reflection example.

```csharp
using System;
using System.Linq;
using System.Reflection;

public interface IPlugin
{
    string Name { get; }
    void Execute();
}

public static class PluginRunner
{
    public static void RunPlugins(Assembly assembly)
    {
        var pluginTypes = assembly
            .GetTypes()
            .Where(t => typeof(IPlugin).IsAssignableFrom(t)
                        && !t.IsInterface
                        && !t.IsAbstract);

        foreach (Type pluginType in pluginTypes)
        {
            if (Activator.CreateInstance(pluginType) is IPlugin plugin)
            {
                Console.WriteLine($"Running {plugin.Name}");
                plugin.Execute();
            }
        }
    }
}
```

What this tests:

- assembly scanning
- interface detection
- runtime object creation
- dynamic system design

Why it is hard:

- you must understand `Assembly`, `Type`, and `Activator`
- you must filter types correctly
- you must think about runtime failures like missing constructors and load errors

## Important Reflection APIs

Useful APIs to remember:

- `typeof(MyType)`
- `obj.GetType()`
- `Assembly.Load(...)`
- `assembly.GetTypes()`
- `type.GetProperties()`
- `type.GetMethods()`
- `type.GetFields()`
- `type.GetConstructors()`
- `type.GetCustomAttributes()`
- `Activator.CreateInstance(type)`
- `methodInfo.Invoke(...)`
- `propertyInfo.GetValue(...)`
- `propertyInfo.SetValue(...)`

## Reflection and Generics

Reflection can also inspect generic types.

Example:

```csharp
Type type = typeof(List<int>);

Console.WriteLine(type.IsGenericType);                 // true
Console.WriteLine(type.GetGenericTypeDefinition());    // List<T>
Console.WriteLine(type.GetGenericArguments()[0].Name); // Int32
```

## Advantages

- Flexible runtime behavior
- Good for frameworks and reusable libraries
- Supports metadata-driven design
- Useful when types are discovered dynamically

## Disadvantages

- Slower than direct code
- More runtime errors if used carelessly
- Harder to refactor safely
- Private member access can break encapsulation
- Can make code harder to understand

## Common Interview Mistakes

- Saying reflection is used in normal business code everywhere
- Ignoring performance cost
- Confusing reflection with serialization
- Forgetting that reflection works with metadata at runtime
- Using reflection when a normal interface or generic solution is better

## Quick Interview Summary

If asked for a short answer:

"Reflection in C# allows runtime inspection of assemblies, types, methods, properties, and attributes. It is useful for frameworks, DI containers, serializers, plugin systems, and metadata-driven behavior. It is powerful, but slower and less type-safe than direct code, so it should be used carefully."

## References

- [Microsoft Learn - Reflection in .NET](https://learn.microsoft.com/en-us/dotnet/fundamentals/reflection/overview)
- [Microsoft Learn - Attributes and reflection](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/)
- [Microsoft Learn - Generics and reflection](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/generics-and-reflection)
