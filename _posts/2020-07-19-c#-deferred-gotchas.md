---
title: C# Deferred Gotchas
date: Y2020-07-19 22:40:20 +0800
categories: [Tech, Csharp]
tags: [Csharp]
---

Linq? IEnumerable? We use it everyday for good reasons, performance, abstraction, developer productivity and etc. You name it. Normally everything works well until the "gotchas" of Linq and IEnumerable takes you into hours of "phantom" production debugging. The reason is because it does not produce any errors and code execution does not get interrupted. In this post, I'll try to explain the deferred nature of Linq and IEnumerable so we can be more aware when we use them and prevent "phantom" bugs.

# The Gotchas
Let's start with a simple scenario to showcase the deferred "gotchas" of Linq and IEnumerable. We will have a simple `Person` model and a method that returns a list of `Person`, something similar to a repository. After returning the list of `Person` we populate its `FullName` property in two (2) ways:

- Use Linq's `Select` to assign the `FirstName` and `LastName` to the `FullName` property.
- Iterate over the list using `foreach` and assign the `FullName` property to `null`.

These happens in sequence. Lastly, a check will throw an error if the `FullName` property is `null`, otherwise it is displayed in the console. The check will highlight the "gotchas".

## Code
### Person.cs
```csharp
public class Person
{
    public string FirstName { get; set; }

    public string LastName { get; set; }

    public string FullName { get; set; }
}
```

### PersonRepository.cs
```csharp
public class PersonRepository
{
    public IEnumerable<Person> GetPersons() =>
        new List<Person>
        {
            new Person { FirstName = "Hello", LastName = "World" }
        }.AsQueryable();
}
```
_Note: We'll address later the need to cast the list using `.AsQueryable()`. Esentially this is to replicate that the data came from a database._

### Program.cs
```csharp
public static void Main()
{
    var repository = new PersonRepository();
    var persons = repository.GetPersons();

    persons = persons.Select(p =>
    {
        p.FullName = $"{p.FirstName} {p.LastName}";
        return p;
    });

    foreach (var p in persons)
    {
        p.FullName = null;
    }

    foreach (var p in persons)
    {
        _ = p.FullName ?? throw new NullReferenceException();
        Console.WriteLine(p.FullName);
    }

    Console.ReadKey();
}
```

## Test
Just from looking at the code we know it will throw an error since it is sequential, the assignment of `null` was the last operation before the check. 

- __Gotcha #1: The code actually prints "Hello World" in the console.__

Let's make the `IEnumerable<Person>` into a list `var persons = repository.GetPersons().ToList();`.

- __Gotcha #2: Now the code throws an error.__

Pretty weird right? We have two behaviors on sequential code that looks familiar at first glance. We'll explore now the reason behind these.

# The Why
Deferred execution or lazy evaluation what does it mean? To put it simply it means that the code does not get "runned" in the current time or as we expected based on code structure. This is the reason why the code above seems to skip some code that we expect to have some kind of effect. There are two (2) items that are lazily evaluated `IEnumerable` and `Select` from the code above.

- IEnumerable - items inside does not mean that there is data in memory. IEnumerable is more of a promise that something is in memory. We can exhibit this from the note above about the `AsQueryable()` extension. What it does is that it deferrs the storage of the items in memory for optimized query execution.

Peeking the variable at runtime it shows `Expanding the Results View will enumerate the the IEnumerble` in other words materialize it to reality.
// Picture

Removing the `AsQueryable()` extension essentialy making it a list and peeking at the variable now shows its count and contents. The list is now concrete and in memory. 
// Picture

- Linq - operations are executed when they are needed. Generally Linq extensions follows the behavior of the list its querying on. The Linq operations are deferred until the list is materialized.
// Picture

Here the list is materialized using the `.ToList()` extension.
// Picture

# Conclusion
We saw the "gotchas" of deferred execution or lazy evaluation can cause "phantom" bugs. We also explored the reasons behind it. The example is trivial we can easily debug it, in the production environment these pieces of code are seperated in classes or even projects which is much much harder to debug and locate. 

How can we assure that the code we write are evaluated as we expect them to be? We can actually follow the rule of thumb we use in `async/await` in working in deferred nature. 

> In `async` code we `await` something when we need the value now before proceeding in the code. On the otherhand, in `IEnumerable` or `Linq` we can `.ToList()` something when we need the value now before proceeding in the code.
