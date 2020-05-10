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

## Mediator - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Mediator)
### When
- Complex interactions happen between several objects.
- Objects cannot be reused due to the said interactions.
- Customizing functionality without much inheritance.

### Effects
- Reduces inheritance by localizing behavior.
- Decouples object interaction, allowing it to vary independently.
- Simplifies object interaction, optimally to one-to-many compared to many-to-many interactions.
- Centralizes object interaction, making it complex and monolithic.

### Relationships
- The Mediator pattern is often compared to the __Facade__ pattern, Facade encapsulates subsystems while Mediator encapsulates object communiation.
- Objects can communicate with the Mediator using the __Observer__ pattern.

## Memento - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Memento)
### When
- Storing (snapshot) object state.

### Effects
- Enforces encapsulation boundaries.
- Expensive use of memory.

### Relationships
- The __Command__ pattern usually uses Memento's to store a history of commands.
- Memento's state history can be traversed by the __Iterator__ pattern.

## Observer - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Observer)
### When
- Objects interaction are structured in publish-subscribe.
- Changes ripples to multiple objects.
- Detection of changes needs to be encapsulated.

### Effects
- Objects subscribed to changes are decoupled from objects publishing the changes.
- Broadcasting support.
- Changes can be hard to track down since it happens unexpectedly.

### Relationships
- The communication of the Observer pattern can be encapsulated by the __Mediator__ pattern.
- The instance of the Observer's communication can be represented by the __Singleton__ pattern.

## State - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/State)
### When
- Object behavior depends on its state.
- Conditional statements influence object behavior changes.

### Effects
- Encapsulates and modularizes state specific behavior.
- Transitions are defined explicitly.
- Reusability of object behavior.

### Relationships
- States can be shared optimally using the __Flyweight__ pattern.
- Due to States only containing behavior, they are usually represented by the  __Singleton__ pattern.

## Strategy - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Strategy)
### When
- Classes only contains behavior/algorithm.
- Behaviors/algorithms needs to be encapsulated.
- Hide sensitive data needed by the behavior/algorithm.
- Multiple conditional statements only contains behaviors/algorithms.

### Effects
- Provides an alternative to inheritance that is focused to abstracting behaviors.
- Conditional statements are eliminated.
- Knowledge of the strategies are now needed by the client.
- Interface overhead since some strategies are simpler than others.
- Adds complexity for behavior abstraction.

### Relationships
- Since strategies only contain behavior, they can be constructed by the __Flyweight__ pattern.

## Template Method - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Template)
### When
- Only a section of the behavior changes.
- Factoring out common behavior.
- Controlling inheritance extension by providing "hooks" at specified points.

### Effects
- Inversion of control.
- Provides an alternative to class overriding.

### Relationships
- The Template Method pattern is often compared with the __Strategy__ pattern. Template Method varies only a section of the bahavior while Strategy varies the whole behavior.
- Template Methods are used to invoke __Factory Methods__.

I hope the series did not teach you anything.... well in someway. The goal of the series was never to "teach", actually the goal is pretty similar to the original "[Gang of Four](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8)" book, which was to __list__ the Design Patterns used by the industry. The series aimed to show the different aspects of the Design patterns which is not commonly talked about. I think these aspects are very important __before__ learning to the apply the Design Patterns. We as developers love to jump in right into the code, well for one thing it's more fun. But as a "Profesional Developer" we need to understand that sample code is not enough, we need to consider the whole picture to better formulate an elegant, robust, and succint solution. By providing these "Secrets" I aim to show the other half of the picture so we can deeply understand what a Design Pattern really is and not naively implementing it.