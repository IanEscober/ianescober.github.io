---
title: Design Pattern Secrets - Creational
date: Y2020-01-14 15:46:20 +0800
categories: [Tech, C#]
tags: [C#, Design Patterns, Series]
---

Design Patterns we know it, we use it, it is everywhere. If so happens you have not come across it, there are numerous blogs, videos, and books where you can learn about Design Patterns. I personally recommend the original book by the [Gang of Four](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612) and it will also be the basis of this series. Now, if there are tons of resources out there, books written in depth, amazing courses, etc. Why am i writing yet another blog post about Design Patterns? Well, I have yet seen a post where it __summarizes the Design Pattern's:__
- __When__
- __Effects__
- __Relationships__

I want this series to be a concise catalog of the Design Patterns with the categories above. There will be no explanation about and how Design Patterns are implemented, as earlier said there are numerous and numerous of resources everywhere. But I would be linking my examples at GitHub. Without further ado, let us begin with Creational Patterns.

# Creational Patterns
## Abstract Factory - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/AbstractFactory)
### When
- A system needs to seperate the creation, composition, and represetation of its products.
- A system produces products from various groups.
- A group of related products is used and needs to be together.
- You want to hide the implementation of a library of products.
### Effects
- It encapsulates concrete classes.
- It simplifies the product groups replacement. 
- It enforces consistency among products.
- It adds difficulty to supporting new product groups.
### Relationships
- Often implemented with __Factory Method__
- Often implemented with __Prototype__
- A concrete factory is often a __Singleton__

## Builder - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Builder)
### When
- The algorithm for creating a complex object should be independent of the parts that make up the object and how they're assembled.
- The construction process must allow different representations for the object that's constructed.
### Effects
- It lets you vary a product's internal representation.
- It isolates code for construction and representation.
- It gives you finer control over the construction process.
### Relationships
- __Abstract Factory__ emphasizes families of product object. Builder focuses on constructing a complex object step by step.
- A __Composite__ is what the builder often builds.

## Factory Method - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/FactoryMethod)
### When
- A class can't anticipate the class of objects it must create.
- A class wants its subclasses to specify the objects it creates.
- Classes delegate responsibility to one of several helper subclasses, and you want to localize the knowledge of which helper subclass is the delegate.
### Effects
- Provides hooks for subclasses
- Connects parallel class hierarchies.
### Relationships
- __Abstract Factory__ is often implemented with Factory Methods.
- Factory Methods are usually called within __Template Methods__.
- __Prototypes__ don not require subclassing Creator. While Factory Method does not require such an operation.

## Prototype - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Prototype)
### When
- When the classes to instantiate are specified at run-time, for example, by dynamic loading; or
- To avoid building a class hierarchy of factories that parallels the class hierarchy of products; or
- When instances of a class can have one of only a few different combinations of state. It may be more convenient to install a corresponding number of prototypes and clone them rather than instantiating the class manually, each time with the appropriate state.
### Effects
- Adding removing products at run-time.
- Specifying new objects by varying values.
- Specifying new objects by varying structure.
- Reduced subclassing.
- Configuring an application with classes dynamically.
### Relationships
- An __Abstract Factory__ might store a set of prototypes from which to clone and return to product objects.
- __Composite__ and __Decorator__ can benefit from Prototype as well.

## Singleton - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Singleton)
### When
- There must be exactly one instance of a class, and it must be accessible to clients from a well-known access point.
- When the sole instance should be extensible by subclassing, and clients should be able to use an extended instance without modifying their code.
### Effects
- Controlled access to sole instance.
- Reduced name sapce.
- Permits refinement of operations and representation.
- Permits a variable number of instances.
- More flexible than class operations.
### Relationships
- These patterns can be implemented in Singleton: __Abstract Factory, Builder,__ and __Prototype__.