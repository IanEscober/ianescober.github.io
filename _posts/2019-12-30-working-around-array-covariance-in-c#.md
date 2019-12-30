---
title: Working Around Array Covariance in C#
date: Y2019-12-30 11:25:20 +0800
categories: [Tech, C#]
tags: [C#, Design Patterns]
---

Covariance and contravariance are esoteric topics in the programming world. Upon hearing them for the first time we think of them as arcane knowledge with complex mathematics bounded with it. Partially this is true, see [Category Theory](https://plato.stanford.edu/entries/category-theory/), but they are actually used in our everyday programming. The moment you create abstractions in your code, covariance or contravariance or both are applied. In essence:
 - __Covariance__ – is the assignment of __derived__ types __to__ its __parent__ types.
 - __Contravariance__ – is essentially the reverse of covariance, passing of __parent__ types __to__ __derived__ types.

To have a better grasp of the situation, let us explain them in language we can naturally understand - in code. The Mediator Design Pattern [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Mediator) will provide us enough abstractions to highlight array covariance. The example are composed of a generic `IPayload` and `IHandler`. Without going into much detail, a payload is the model of the data passed to the handler. The handlers accept a specific derived type of the payload for them to process specifically. The nomenclature are based from [CQRS](https://martinfowler.com/bliki/CQRS.html).
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

 // Contravariance
 Action<IPayload> actionPayload = payload => Console.WriteLine("A payload");
 Action<QueryPayload> contravariantPayload1 = actionPayload;    // Parent to Derived
 Action<CommandPayload> contravariantPayload2 = actionPayload;  // Parent to Derived
 ```

# Array Covariance Issue

