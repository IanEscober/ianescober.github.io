---
title: Design Pattern Secrets - Behavioral
date: Y2020-03-13 16:14:20 +0800
categories: [Tech, Csharp]
tags: [Csharp, Design Patterns, Series]
---

We are now on the last category of the Design Patterns which is the __Behavioral Patterns__, concluding this series. The "Secrets" of the previous two categories of this series can be found at:
 - [Creational Patterns](https://ianescober.github.io/posts/design-patterns-secrets-creational/)
 - [Structural Patterns](https://ianescober.github.io/posts/design-patterns-secrets-strcutural/)

As always the catalog would be again structured in:
- __When__
- __Effects__
- __Relationships__

# Structural Patterns
## Chain of Responsibility (Pipeline) - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Pipeline)
### When
- A request needs to be handled multiple times.
- The handler of the request are needed to be varied dynamically.
- The handler of the request can be seperated into "steps".

### Effects
- Requests Handlers are transparent to the senders, reducing coupling.
- Adding responsibilty to handling requests are easily extensible.
- Due to the nature of the handlers transparency to the requests, there is no assurance of the request being handled.
- Logical "steps" of handling a request is materialized.

### Relationships
- The Pipeline can be structured with the __Composite__ pattern.

## Command - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Command)
### When
- Request "callback" or action needs be encapsulated.
- A timeline of actions can to be recorded or queued for tracking and deffered performance.
- Reversing or "undo" mechanism for actions is needed.
- Request equates to a transaction.

### Effects
- Seperates the logic of handling requests or callbacks from the invoker.
- Commands can be structured into hierarchies which reflects the process flow.

### Relationships
- Command hierarchies can be crafted with the __Composite__ pattern.
- Command histories is ecapsulated by the __Memento__ pattern.
- For space optimization, the __Prototype__ pattern can create clones of Commands.

<!-- Iterpreter Pattern -->

## Iterator - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Iterator)
### When
- Provide a way to access sensitive data inside aggregates without exposing it.
- Generalizing a pattern for traversing aggregates.
- Supporting multiple traversal techniques.

### Effects
- Swapping traversal techniques in aggregates are simpler.
- Aggregates logic are simplified since iteration logic is abstracted out in the Iterator.
- Different iteration techniques can now be done concurrently.

### Relationships
- Iterations state can be stored by the __Memento__ pattern.
- The traversing logic can be abstracted with the __Template Method__ pattern.

I hope the series did not teach you anything.... well in someway. The goal of the series was never to "teach", actually the goal is pretty similar to the original "[Gang of Four](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8)" book, which was to __list__ the Design Patterns used by the industry. The series aimed to show the different aspects of the Design patterns which is not commonly talked about. I think these aspects are very important __before__ learning to the apply the Design Patterns. We as developers love to jump in right into the code, well for one thing it's more fun. But as a "Profesional Developer" we need to understand that sample code is not enough, we need to consider the whole picture to better formulate an elegant, robust, and succint solution. By providing these "Secrets" I aim to show the other half of the picture so we can deeply understand what a Design Pattern really is and not naively implementing it. Also, having it written in a blog allowed it to be ubiquitously available for everyone.