# C# Reflection Interview Questions and Answers

These are common reflection questions for C# and .NET interviews, grouped by difficulty and aligned with Microsoft documentation.

## Easy

1. What is reflection in C#?
   Answer: Reflection is the ability to inspect metadata about assemblies, types, methods, properties, fields, and attributes at runtime.
2. Which namespace is mainly used for reflection?
   Answer: `System.Reflection`.
3. What is a `Type` object?
   Answer: A `Type` object represents metadata about a type, such as its name, members, base type, and implemented interfaces.
4. How do you get a `Type` in C#?
   Answer: Common ways are `typeof(MyType)`, `obj.GetType()`, and `Type.GetType(...)`.
5. What is the difference between `typeof()` and `GetType()`?
   Answer: `typeof()` is used when the type is known at compile time. `GetType()` is used on an object instance at runtime.
6. What can reflection inspect?
   Answer: It can inspect assemblies, classes, interfaces, methods, properties, fields, constructors, parameters, attributes, and generic type information.
7. What is `Assembly` in reflection?
   Answer: It represents a compiled .NET unit such as a DLL or EXE and can be inspected for the types it contains.
8. What is `MethodInfo`?
   Answer: `MethodInfo` contains metadata about a method and can also be used to invoke that method dynamically.
9. What is `PropertyInfo`?
   Answer: `PropertyInfo` represents metadata about a property and lets you read or write its value dynamically.
10. Why is reflection useful?
    Answer: It is useful when code must inspect or use types dynamically, such as in frameworks, serializers, ORMs, plugin systems, and test tools.

## Medium

1. How do you invoke a method using reflection?
   Answer: Use `GetMethod` to get a `MethodInfo`, then call `Invoke` with the target instance and parameters.
2. How do you create an object dynamically with reflection?
   Answer: Use `Activator.CreateInstance(type)` or a constructor obtained through reflection.
3. How do you get all properties of a type?
   Answer: Use `type.GetProperties()`.
4. How do you read custom attributes with reflection?
   Answer: Use methods such as `GetCustomAttributes()` on a type or member.
5. What is the difference between early binding and late binding?
   Answer: Early binding resolves members at compile time. Late binding resolves them at runtime, often using reflection.
6. What are common real-world uses of reflection?
   Answer: Dependency injection, ORM mapping, JSON/XML serialization, plugin loading, validation, model binding, and test discovery.
7. Can reflection access private members?
   Answer: Yes, in many cases, if the appropriate binding flags are used, though this should be done carefully.
8. What are binding flags?
   Answer: `BindingFlags` control which members reflection returns, such as public, non-public, instance, or static members.
9. How do you inspect whether a type implements an interface?
   Answer: Use checks such as `typeof(IMyInterface).IsAssignableFrom(type)`.
10. What is the performance impact of reflection?
    Answer: Reflection is slower than direct strongly typed code because it resolves metadata and invokes members dynamically at runtime.

## Hard

1. What are the disadvantages of reflection?
   Answer: It is slower, less type-safe, harder to refactor, and can weaken encapsulation if it accesses private members.
2. When should reflection be avoided?
   Answer: Avoid it in hot paths or when normal interfaces, generics, or direct strongly typed code can solve the problem more clearly.
3. How would you build a plugin system with reflection?
   Answer: Load an assembly, scan its types, filter implementations of a shared interface, create instances dynamically, and execute them.
4. How can reflection and attributes be used together?
   Answer: Attributes add metadata to code, and reflection reads that metadata at runtime to drive behavior such as validation or mapping.
5. What is the difference between reflection and source generation?
   Answer: Reflection works at runtime and is dynamic. Source generation produces code at build time and can be faster and safer.
6. How can reflection inspect generic types?
   Answer: Use APIs like `IsGenericType`, `GetGenericArguments()`, and `GetGenericTypeDefinition()`.
7. What is the difference between `Assembly.Load` and `Assembly.LoadFrom` in practice?
   Answer: Both load assemblies, but `LoadFrom` uses a file path and is common in plugin scenarios, while `Load` uses the assembly identity.
8. What risks exist when invoking methods by reflection?
   Answer: Missing methods, wrong parameter types, runtime exceptions, null results, and weaker compile-time safety.
9. Why do frameworks often cache reflection results?
   Answer: To reduce repeated metadata lookup cost and improve performance.
10. What is a better alternative to reflection in some modern .NET scenarios?
    Answer: Interfaces, generics, delegates, expression trees, or source generators can often provide clearer and faster solutions.

## Very Common Coding Questions

1. Write code to print all property names of an object.
   Answer:

```csharp
public static void PrintPropertyNames(object obj)
{
    Type type = obj.GetType();

    foreach (var property in type.GetProperties())
    {
        Console.WriteLine(property.Name);
    }
}
```

2. Write code to call a method dynamically using reflection.
   Answer:

```csharp
Type type = typeof(Calculator);
object? instance = Activator.CreateInstance(type);
MethodInfo? method = type.GetMethod("Add");
object? result = method?.Invoke(instance, new object[] { 5, 7 });
```

3. Write code to read a property value dynamically.
   Answer:

```csharp
PropertyInfo? property = obj.GetType().GetProperty("Name");
object? value = property?.GetValue(obj);
```

4. Write code to find all classes in an assembly that implement an interface.
   Answer:

```csharp
var types = assembly
    .GetTypes()
    .Where(t => typeof(IPlugin).IsAssignableFrom(t)
                && !t.IsInterface
                && !t.IsAbstract);
```

## .NET Core Follow-Up Questions

1. Where is reflection commonly used in ASP.NET Core or .NET libraries?
   Answer: It is commonly used in dependency injection, model binding, validation, serialization, middleware discovery, and plugin architectures.
2. Why is reflection important for custom attributes in .NET?
   Answer: Because attributes only become useful at runtime when reflection reads them and applies behavior based on their metadata.
3. Can reflection work with generic types in .NET Core?
   Answer: Yes. Reflection can inspect generic type definitions, type arguments, and generic method information.
4. Why do performance-sensitive frameworks reduce reflection usage after startup?
   Answer: Because repeated reflection can be expensive, so many frameworks discover metadata once and then cache delegates or compiled accessors.
5. Is reflection always the best choice for extensibility?
   Answer: No. Reflection is powerful, but interfaces, generics, and explicit contracts are often simpler and safer if the types are already known.

## Recommended Answer Style

For interview answers:

- Start with runtime metadata inspection
- Mention one or two real uses such as DI, serialization, or plugins
- Mention one risk such as performance or reduced type safety
- If asked for depth, explain attributes and dynamic invocation

## Sources

Recurring interview-question patterns were taken from:

- [We Create Problems - C# Interview Questions](https://www.wecreateproblems.com/interview-questions/c-sharp-interview-questions)
- [C# Corner - C# Interview Questions](https://www.c-sharpcorner.com/UploadFile/puranindia/C-Sharp-interview-questions/)

Technical alignment was checked against:

- [Microsoft Learn - Reflection in .NET](https://learn.microsoft.com/en-us/dotnet/fundamentals/reflection/overview)
- [Microsoft Learn - Attributes and reflection](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/)
- [Microsoft Learn - Generics and reflection](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/generics-and-reflection)
