---
title: Design Pattern Secrets - Creational
date: Y2020-01-14 15:46:20 +0800
categories: [Tech, Csharp]
tags: [Csharp, Design Patterns, Series]
seo:
  date_modified: 2020-01-14 15:46:20 +0800
---

![Building Blocks](https://drive.google.com/uc?export=view&id=18b0G0agdx-vBMgL23EKEIbSE7LqA9iph)
Design Patterns we know it, we use it, it is everywhere. If so happens you have not come across it, there are numerous blogs, videos, and books where you can learn about Design Patterns. I personally recommend the original book by the [Gang of Four](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612) and it will also be the basis of this series. Now, if there are tons of resources out there, books written in depth, amazing courses, etc. Why am i writing yet another blog post about Design Patterns? Well, I have yet seen a post where it __summarizes the Design Pattern's:__
- __When__
- __Effects__
- __Relationships__

I want this series to be a concise catalog of the Design Patterns with the categories above. There will be no explanation about and how Design Patterns are implemented, as earlier said there are numerous and numerous of resources everywhere. But I would be linking my [examples](http://github.com/ianescober/designpatterns) at GitHub. Without further ado, let us begin with Creational Patterns.

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
- The building logic of complex objects must be seperated.
- The building process have multiple representation of the complex object being constructed.

### Effects
- It seperates the products representation.
- It encapsulates building and representation seperately.
- It provides fine-grained control on the building process.

### Relationships
- __Abstract Factory__ focuses on creating families of products. Builder focuses on building a complex object in a step by step manner.
- A __Composite__ is what the builder often builds.

## Factory Method - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/FactoryMethod)
### When
- A class have no knowledge the objects to be created.
- A class wants to delegate the object creation to its subclasses.
- Classes pass the process to other helper subclasses, and you want to encapsulate that helper subclass.

### Effects
- Eliminates the need to bind concrete classes.
- Allows extensions of an object by providing a hook.
- Connects parallel class hierarchies.
- Clients might have to subclass a "Creator" class in order to create a particular concrete class.

### Relationships
- __Abstract Factory__ is implemented with Factory Methods.
- Factory Methods are usually executed within __Template Methods__.
- __Prototypes__ do not require subclassing Creator. While Factory Method does not require such an operation.

## Prototype - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Prototype)
### When
- The classes are instantiated dynamically (run-time).
- You want to avoid a hierarchy of factories that matches the depth of the product it creates.
- There are only a few combinations of state a class can have.

### Effects
- Encapsulates concrete types to reduce client knowledge (coupling).
- Work with application-specific classes without alteration.
- Dynamically (run-time) add or remove objects.
- Create new objects by registering subclasses as prototype. 
- Create new objects by reusing (cloning) subclasses. 
- Reduced subclassing.
- Configuring an application with classes dynamically.

### Relationships
- An __Abstract Factory__ can store a set of prototypes to clone from.
- __Composite__ and __Decorator__ can benefit from Prototype as well.

## Singleton - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Singleton)
### When
- Only one instance of class is needed and it must be globally known.
- The sole instance is extensible by subclassing and should not affect clients after.

### Effects
- Controlled access to sole instance.
- Reduced name space.
- Allows to dynamically load objects as well as extension.
- Allows a variable number of instances (Multiton).
- More flexible than class operations.

### Relationships
- These patterns can be implemented in Singleton: __Abstract Factory, Builder,__ and __Prototype__.