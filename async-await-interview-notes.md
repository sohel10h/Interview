# Async and Await in C# — Interview Notes

## 1. What problem do `async` and `await` solve?

`async` and `await` help us write **asynchronous code** in a way that looks similar to normal synchronous code.

We use them when an operation may take time, such as:

- database calls
- HTTP/API calls
- file read/write
- network calls
- waiting for timers
- message queue operations

The goal is usually **not to make the operation faster**, but to make the program **more responsive** and use threads more efficiently.

---

## 2. Simple definition

### `async`
`async` marks a method as asynchronous.

Example:

```csharp
public async Task<string> GetDataAsync()
{
    ...
}
```

### `await`
`await` tells the method:

- start the async operation
- pause this method here
- return control to the caller
- continue later when the awaited task finishes

---

## 3. Return types of async methods

Common return types:

```csharp
Task
Task<T>
void
```

### `Task`
Use when the method does not return a value.

```csharp
public async Task SaveAsync()
{
    await Task.Delay(1000);
}
```

### `Task<T>`
Use when the method returns a value.

```csharp
public async Task<string> GetNameAsync()
{
    await Task.Delay(1000);
    return "Sohel";
}
```

### `async void`
Avoid it except for event handlers.

```csharp
private async void Button_Click(object sender, EventArgs e)
{
    await Task.Delay(1000);
}
```

Why avoid `async void`?

- caller cannot await it
- exceptions are harder to catch
- difficult to test

---

## 4. What actually happens when `await` is used?

Consider:

```csharp
public async Task DoWorkAsync()
{
    Console.WriteLine("A");
    await Task.Delay(1000);
    Console.WriteLine("B");
}
```

### Step by step

1. method starts
2. prints `A`
3. reaches `await Task.Delay(1000)`
4. if the task is not completed, method pauses there
5. control returns to caller
6. after 1 second, method resumes
7. prints `B`

So `await` does **not block the thread** like `Thread.Sleep()` does.

---

## 5. Your scenario: why blank line is printed

Code:

```csharp
class Program {
  private static string result;

  static void Main() {
    SaySomething();
    Console.WriteLine(result);
  }

  static async Task<string> SaySomething() {
    await Task.Delay(5);
    result = "Hello world!";
    return "Something";
  }
}
```

> Note: in the original sample, `return “Something”;` used curly quotes and would not compile.  
> Here we assume normal quotes are used.

### Step by step

#### Step 1
`result` is declared but not assigned:

```csharp
private static string result;
```

Default value of a `string` field is:

```csharp
null
```

#### Step 2
`Main()` starts:

```csharp
SaySomething();
Console.WriteLine(result);
```

#### Step 3
`SaySomething()` is called.

It starts running until it reaches:

```csharp
await Task.Delay(5);
```

#### Step 4
`Task.Delay(5)` is not complete yet, so the method pauses.

That means this line has **not run yet**:

```csharp
result = "Hello world!";
```

#### Step 5
Because `Main()` did **not await** `SaySomething()`, it continues immediately to:

```csharp
Console.WriteLine(result);
```

#### Step 6
At that moment `result` is still `null`.

So:

```csharp
Console.WriteLine(result);
```

prints a **blank line**.

### Why blank line and not the word `null`?
Because `Console.WriteLine(null)` does not print `"null"`.  
It prints an empty line.

---

## 6. Correct version of the above

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    private static string result;

    static async Task Main()
    {
        await SaySomething();
        Console.WriteLine(result);
    }

    static async Task<string> SaySomething()
    {
        await Task.Delay(5);
        result = "Hello world!";
        return "Something";
    }
}
```

Now `Main()` waits until `SaySomething()` finishes, so output becomes:

```text
Hello world!
```

---

## 7. `Task.Delay()` vs `Thread.Sleep()`

### `await Task.Delay(1000)`
- non-blocking
- releases the thread
- good for async code

### `Thread.Sleep(1000)`
- blocking
- thread is stuck doing nothing
- usually bad in async flows

Example:

```csharp
await Task.Delay(1000);
```

This pauses the method, but the thread can do other work.

Example:

```csharp
Thread.Sleep(1000);
```

This blocks the current thread for 1 second.

### Interview one-liner
`Task.Delay` is asynchronous waiting, but `Thread.Sleep` is synchronous blocking.

---

## 8. Example: `await` vs no `await`

### Example without await

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task DoWorkAsync()
    {
        await Task.Delay(1000);
        Console.WriteLine("Work done");
    }

    static void Main()
    {
        DoWorkAsync();
        Console.WriteLine("Main finished");
    }
}
```

### Possible output

```text
Main finished
```

Maybe `"Work done"` may not appear before process exits.

### Why?
Because `Main()` does not wait for `DoWorkAsync()`.

---

### Correct version

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task DoWorkAsync()
    {
        await Task.Delay(1000);
        Console.WriteLine("Work done");
    }

    static async Task Main()
    {
        await DoWorkAsync();
        Console.WriteLine("Main finished");
    }
}
```

### Output

```text
Work done
Main finished
```

---

## 9. Example: multiple async tasks

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task<int> GetValue1Async()
    {
        await Task.Delay(1000);
        return 10;
    }

    static async Task<int> GetValue2Async()
    {
        await Task.Delay(1000);
        return 20;
    }

    static async Task Main()
    {
        var t1 = GetValue1Async();
        var t2 = GetValue2Async();

        var result1 = await t1;
        var result2 = await t2;

        Console.WriteLine(result1 + result2);
    }
}
```

### Output

```text
30
```

### Why is this good?
Both tasks start early, so they can run concurrently.

---

## 10. Sequential await vs concurrent start

### Sequential

```csharp
var a = await GetValue1Async();
var b = await GetValue2Async();
```

This waits for first task to finish, then starts the second.

### Better when independent

```csharp
var task1 = GetValue1Async();
var task2 = GetValue2Async();

var a = await task1;
var b = await task2;
```

or:

```csharp
await Task.WhenAll(task1, task2);
```

### Interview point
If operations are independent, start them first, then await them.

---

## 11. Example with `Task.WhenAll`

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task<string> CallApi1Async()
    {
        await Task.Delay(1000);
        return "API1";
    }

    static async Task<string> CallApi2Async()
    {
        await Task.Delay(1000);
        return "API2";
    }

    static async Task Main()
    {
        var t1 = CallApi1Async();
        var t2 = CallApi2Async();

        var results = await Task.WhenAll(t1, t2);

        foreach (var item in results)
        {
            Console.WriteLine(item);
        }
    }
}
```

---

## 12. Common interview pitfalls

### Pitfall 1: forgetting to await

```csharp
DoSomethingAsync();
```

If the result matters, use:

```csharp
await DoSomethingAsync();
```

---

### Pitfall 2: using `.Result` or `.Wait()`

```csharp
var data = GetDataAsync().Result;
GetDataAsync().Wait();
```

These can:
- block thread
- reduce scalability
- cause deadlock in some environments

---

### Pitfall 3: `async void`
Use only for event handlers.

Bad:

```csharp
public async void ProcessAsync()
{
    await Task.Delay(1000);
}
```

Better:

```csharp
public async Task ProcessAsync()
{
    await Task.Delay(1000);
}
```

---

### Pitfall 4: mixing blocking and async code badly

Bad:

```csharp
public async Task DoAsync()
{
    Thread.Sleep(5000);
}
```

This method is marked async but still blocks the thread.

---

## 13. Deadlock example

Deadlock is a very common interview topic with `async`/`await`.

### Important note
This deadlock usually happens in environments with a synchronization context, like:
- old ASP.NET
- WinForms
- WPF
- UI thread apps

It usually does **not** happen the same way in normal .NET console apps.

### Deadlock code

```csharp
public string GetData()
{
    return GetDataAsync().Result;
}

public async Task<string> GetDataAsync()
{
    await Task.Delay(1000);
    return "Done";
}
```

### Why can this deadlock?

#### Step by step

1. `GetData()` calls `GetDataAsync().Result`
2. `.Result` blocks the current thread
3. `GetDataAsync()` starts
4. it reaches `await Task.Delay(1000)`
5. after delay, it tries to continue on the original synchronization context
6. but that thread/context is blocked by `.Result`
7. async continuation waits for the thread
8. blocked thread waits for async result
9. both wait for each other → deadlock

### Simple explanation
- caller blocks thread waiting for async method
- async method needs same thread to continue
- neither can move

---

## 14. How to avoid deadlock

### Best approach: async all the way

Bad:

```csharp
public string GetData()
{
    return GetDataAsync().Result;
}
```

Good:

```csharp
public async Task<string> GetData()
{
    return await GetDataAsync();
}
```

Then caller also awaits it:

```csharp
var data = await service.GetData();
```

### Another tool: `ConfigureAwait(false)`

```csharp
public async Task<string> GetDataAsync()
{
    await Task.Delay(1000).ConfigureAwait(false);
    return "Done";
}
```

This tells the continuation it does not need the original context.

But for interview answer, the safest statement is:

> Best practice is still async all the way. Do not solve architecture problems only by depending on `ConfigureAwait(false)`.

---

## 15. Real deadlock example for UI / ASP.NET style apps

```csharp
public string LoadText()
{
    return LoadTextAsync().Result;
}

public async Task<string> LoadTextAsync()
{
    await Task.Delay(1000);
    return "Hello";
}
```

In UI or classic ASP.NET apps this may deadlock for the same reason:
the continuation wants to resume on the original context, but that context is blocked by `.Result`.

---

## 16. Why console apps often confuse people

In console apps, people try `.Result` and sometimes it seems to work.

Then they think:

> `.Result` is fine.

But the problem is:
- it still blocks a thread
- it hurts scalability
- in other app types it may deadlock

So in interviews say:

> Even if `.Result` works in some console scenarios, it is still not recommended for async code.

---

## 17. Exception handling with async/await

```csharp
public async Task DoWorkAsync()
{
    try
    {
        await Task.Delay(1000);
        throw new Exception("Something failed");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

Use `try/catch` around awaited calls.

Also:

```csharp
await DoWorkAsync();
```

is better than ignoring the task.

---

## 18. Fire-and-forget warning

Sometimes people do this:

```csharp
_ = SendNotificationAsync();
```

This is called fire-and-forget.

Use carefully because:
- exceptions may be lost
- caller does not know when it finished
- difficult to retry/monitor

This is usually not ideal for important business logic.

Better use:
- background service
- queue
- outbox pattern
- proper logging and retry

---

## 19. CPU-bound vs I/O-bound work

### I/O-bound
Examples:
- DB call
- API call
- file/network operation

Use async/await naturally.

```csharp
var data = await httpClient.GetStringAsync(url);
```

### CPU-bound
Examples:
- heavy calculation
- image processing
- encryption loop
- large data transformation

`async/await` does not magically make CPU work faster.

Sometimes use:

```csharp
await Task.Run(() => HeavyCalculation());
```

But do not overuse `Task.Run` on server-side apps.

### Interview answer
`async/await` is most beneficial for I/O-bound work, not CPU-bound work.

---

## 20. Scenario: API call example

```csharp
public async Task<string> GetUserFromApiAsync()
{
    using var client = new HttpClient();
    var result = await client.GetStringAsync("https://example.com/user");
    return result;
}
```

Why use async here?
Because network call may take time. We do not want to block the thread unnecessarily.

---

## 21. Scenario: database call example

```csharp
public async Task<User> GetUserAsync(int id)
{
    return await _dbContext.Users.FirstAsync(x => x.Id == id);
}
```

In real applications:
- use async DB methods
- improve thread usage
- improve scalability

---

## 22. Scenario: file example

```csharp
public async Task<string> ReadFileAsync(string path)
{
    return await File.ReadAllTextAsync(path);
}
```

Again, this is I/O-bound work, so async is a good fit.

---

## 23. Important interview questions and answers

### Q1. Does `async` create a new thread?
No. Not automatically.

`async`/`await` is about asynchronous continuation, not guaranteed new thread creation.

---

### Q2. Does `await` block the thread?
Usually no. It pauses the method, not the thread.

---

### Q3. Can an async method run synchronously?
Yes. It runs synchronously until it reaches the first incomplete awaited task.

---

### Q4. Why not use `Thread.Sleep()` inside async method?
Because it blocks the thread and defeats the benefit of async waiting.

---

### Q5. Why avoid `.Result` and `.Wait()`?
Because they block, reduce scalability, and can cause deadlocks.

---

### Q6. What is the best practice?
Async all the way.

---

### Q7. When should we use `async void`?
Only for event handlers.

---

### Q8. Is `Task` same as thread?
No.
A `Task` represents asynchronous work. It is not the same thing as a dedicated thread.

---

## 24. Interview-ready comparison table

| Topic | `await Task.Delay(1000)` | `Thread.Sleep(1000)` |
|---|---|---|
| Nature | asynchronous wait | synchronous blocking |
| Blocks thread? | No | Yes |
| Good for async code? | Yes | No |
| Scalable for server apps? | Better | Worse |
| UI responsiveness | Better | Can freeze UI |

---

## 25. Another scenario: mistake with `async void`

Bad:

```csharp
public async void SaveData()
{
    await Task.Delay(1000);
    throw new Exception("Failed");
}
```

Why bad?
Caller cannot await it, and exception handling becomes difficult.

Better:

```csharp
public async Task SaveDataAsync()
{
    await Task.Delay(1000);
    throw new Exception("Failed");
}
```

---

## 26. Another scenario: wrong assumption that async means parallel

Code:

```csharp
var a = await GetAAsync();
var b = await GetBAsync();
```

This is not parallel. This is sequential.

For concurrency:

```csharp
var taskA = GetAAsync();
var taskB = GetBAsync();

await Task.WhenAll(taskA, taskB);
```

---

## 27. Best practices summary

1. Use async for I/O-bound operations
2. Return `Task` or `Task<T>`
3. Avoid `async void` except events
4. Avoid `.Result` and `.Wait()`
5. Use `await` properly
6. Prefer async all the way
7. Do not use `Thread.Sleep()` inside async code
8. Use `Task.WhenAll` for independent async operations
9. Handle exceptions properly
10. Be careful with fire-and-forget

---

## 28. Very short interview summary

You can say:

> `async` and `await` allow us to write non-blocking asynchronous code in a readable way.  
> `await` pauses the method, not the thread, until the task completes.  
> It is most useful for I/O-bound work like API, database, and file operations.  
> Common mistakes are forgetting to await, using `.Result`/`.Wait()`, and using `async void`.  
> `.Result` can even cause deadlock in UI or classic ASP.NET environments.  
> Best practice is async all the way.

---

## 29. Short answer for your original example

If the code is corrected to use normal quotes, then:

```csharp
SaySomething();
Console.WriteLine(result);
```

prints a blank line because:

- `result` starts as `null`
- `SaySomething()` pauses at `await Task.Delay(5)`
- `Main()` does not wait
- `Console.WriteLine(result)` runs before `result = "Hello world!"`

---

## 30. Final interview tip

When answering in interview, always explain:

- what the method returns (`Task` / `Task<T>`)
- whether caller is awaiting or not
- whether thread is blocked or not
- whether continuation resumes later
- whether deadlock risk exists due to `.Result` / `.Wait()`
- whether the work is I/O-bound or CPU-bound

That makes your answer strong and practical.
