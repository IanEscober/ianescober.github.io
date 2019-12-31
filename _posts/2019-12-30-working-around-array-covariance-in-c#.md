---
title: Working Around Array Covariance in C#
date: Y2019-12-30 11:25:20 +0800
categories: [Tech, C#]
tags: [C#, Design Patterns, Hack]
seo:
  date_modified: 2019-12-31 12:29:39 +0800
---

Covariance and contravariance are esoteric topics in the programming world. Upon hearing them for the first time we think of them as arcane knowledge with complex mathematics bounded with it. Partially this is true, see [Category Theory](https://plato.stanford.edu/entries/category-theory/), but they are actually used in our everyday programming. The moment you create abstractions in your code, covariance or contravariance or both are applied. In essence:
 - __Covariance__ – is the assignment of __derived__ types __to__ its __parent__ types.
 - __Contravariance__ – is essentially the reverse of covariance, passing of __parent__ types __to__ __derived__ types.

To have a better grasp of the situation, let us explain them in language we can naturally understand - in code. The Mediator Design Pattern [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Mediator) will provide us enough abstractions to highlight array covariance. The example are composed of a generic `IPayload` and `IHandler`. Without going into much detail, a payload is the model of the data passed to the handler. The handlers accept a specific derived type of the payload for them to process specifically. The nomenclature are based from [CQRS](https://martinfowler.com/bliki/CQRS.html). __The goal is to create a single array of Handlers for both Query and Command Handlers which is currently prevented by Covariance.__
 - Payloads
 ```csharp
 public interface IPayload { } // Marker Interface
 public class QueryPayload : IPayload { ... }
 public class CommandPayload : IPayload { ... }
 ```
 - Handlers
 ```csharp
 public interface IHandler<T> where T : IPayload { ... }
 public class QueryHandler : IHandler<QueryPayload> { ... }
 public class CommandHandler : IHandler<CommandPayload> { ... }
 ```

 - Covariance and Contravariance in Payloads
 ```csharp
 // Covariance
 QueryPayload queryPayload = new QueryPayload { Query = "A Query" };
 CommandPayload commandPayload = new CommandPayload { Command = "A Command", Arguments = new string[] { "Arg 1", "Arg 2" } };
 IPayload covariantPayload1 = queryPayload;     // Derived to Parent
 IPayload covaraintPayload2 = commandPayload;   // Derived to Parent
 ```
 ```csharp
 // Contravariance
 Action<IPayload> actionPayload = payload => Console.WriteLine("A payload");
 Action<QueryPayload> contravariantPayload1 = actionPayload;    // Parent to Derived
 Action<CommandPayload> contravariantPayload2 = actionPayload;  // Parent to Derived
 ```

# Array Covariance Pitfall
Now to make covariance and contravariance more interesting, let us incorporate them with arrays. Arrays in C# are covariant since version 1.0. Allowing us to write code similar from below.
```csharp
object[] array = new String[3];
```
From the previous example, we'll create an array of payloads and handlers to highlight the issue.
```csharp
IPayload[] arrayCovariantPayloads1 = new QueryPayload[] { queryPayload };
IPayload[] arrayCovariantPayloads2 = new CommandPayload[] { commandPayload };

IHandler<IPayload>[] arrayCovariantHandlers1 = new QueryHandler[] { new QueryHandler() };
IHandler<IPayload>[] arrayCovariantHandlers2 = new CommandHandler[] { new CommandHandler() };
```
Covariance for the payloads compiles successfully. But for the handlers it is a different story. An error is produced by the compiler:
> Cannot implicitly convert type 'ArrayCovariance.Handlers.CommandHandler[]' to 'ArrayCovariance.Handlers.IHandler<ArrayCovariance.Payloads.IPayload>[]'. An explicit conversion exists (are you missing a cast?)

# Pitfall or Safety Net?
Let us break down the code to see what is happening. The `QueryHandler` is derived from `IHandler<T>` where `T` is `QueryPayload` which is derived from `IPayload`. Same for `CommandHandler` where `T` is `CommandPayload` instead which is also derived from `IPayload`. Applying covarince we should be able to assing to the variable with type `IHandler<IPayload>`. But making this variables as an array breaks it. Why so?

This actually by design which prevents us from type mismatch errors at run-time. The compiler prevent us from writing code similar from below which is our intent:
```csharp
IHandler<IPayload>[] arrayCovarinatHandlers = new IHandler<IPayload>[] { new QueryHandler(), new CommandHandler() };
```
From earlier we stated that __"the handlers accept a specific derived type of the payload for them to process specifically"__. Having an array of mixed handlers negates this. For example we have a method that accepts an `IHandler<IPayload>[]` and an `IPayload`, which then process the Payload against the Handlers array. If we pass the array from above and a `QueryPayload`, surely an error will be thrown. The compiler is smart enough to prevent the probable processing mishaps at run-time. __But__ let us say we have some sort of type filtering when we process the Payload and having an array of mixed handlers is our intent. How can we bypass Covariance itself?

# The Workaround
Being stubborn developers or rather aware developers, we sometime know better than the compiler and we want to persist our intent. With caution of course. __The solution is to flatten the type to trick the compiler, which can be easily done through a Wrapper.__
```csharp
public abstract class HandlerWrapper
{
    public abstract Task HandleAsync(IPayload payload);
}

public class HandlerWrapperImp<T> : HandlerWrapper where T : IPayload
{ 
    private readonly IHandler<T> handler;

    public HandlerWrapperImp(IHandler<T> handler)
    {
        this.handler = handler;
    }

    public override Task HandleAsync(IPayload payload)
    {
        return this.handler.HandleAsync((T)payload);
    }
}
```

Utilizing the Wrapper from above we can instead write our array of Handlers similar from below. Now we can mix the Handler types into a single array which is cleaner than having two (2) seperate arrays for Query and Command Handlers. 
```csharp
 HandlerWrapperImp<QueryPayload> queryHandler = new HandlerWrapperImp<QueryPayload>(new QueryHandler());
 HandlerWrapperImp<CommandPayload> commandHandler = new HandlerWrapperImp<CommandPayload>(new CommandHandler());
 HandlerWrapper[] handlers = new HandlerWrapper[] { commandHandler, queryHandler };
```

Being knowledgeable about covariance and contravariance is great. Occasionally array covariance can catch us developers off guard and can be either a safety net or a burden, depending on our intent. Either way possessing the knowledge, it can be worked around by a simple Wrapper implementation. 

