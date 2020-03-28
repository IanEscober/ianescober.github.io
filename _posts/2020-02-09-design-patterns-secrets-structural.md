---
title: Design Pattern Secrets - Structural
date: Y2020-02-09 21:47:20 +0800
categories: [Tech, C#]
tags: [C#, Design Patterns, Series]
---

![Hierarchy](https://drive.google.com/uc?export=view&id=1c4WPE0vT_L5GdGMA1C3EmaEq9dWXMw-w)
Continuing from the last post covering the [Creational Design Pattern Secrets](https://ianescober.github.io/posts/design-patterns-secrets-creational/), now we will be covering the __Structural Design Pattern Secrets__. As said before this will be a quick catalog of information that are rarely highlighted about Design Patterns. Examples can be found at my [Github](http://github.com/ianescober/designpatterns). The catalog would be again structured in:
- __When__
- __Effects__
- __Relationships__

# Structural Patterns
## Adapter - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Adapter)
### When
- An interface of a class do not coincide with your needs.
- A need for a common class that connects other incompatible classes with each other.
- A need to use several classes or interfaces without utilizing inheritance.
### Effects
- A single reusable class that can work with many other classes.
- The Adapter class can override some behavior of the classes it is supporting.
- The Adapter class is prone to "doing to much".

### Relationships
- Similar to the __Bridge__ pattern which is about seperating interfaces from implementation while Adapter is about modifying interfaces to make it work with each other.
- Similar to the __Decorator__ pattern which adds funcionality to classes without altering its interface. Supporting recursive implementation, which is not possible with Adapter.
- Similar to the __Proxy__ pattern which creates a "proxy" to make classes work with each other.

## Bridge - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Bridge)
### When
- Coupling of abstraction and implementation must be prevented.
- Abstractions of implementations can be mixed and match or vice versa.
- Changes to abstractions should be transparent to the clients.
### Effects
- Assignment of implementation to abstractions can happen dynamically.
- Implementations and abstractions can be scaled independently.
- Clients are unaffected by changes in implementations and abstractions.
### Relationships
- A Bridge can be create by the __Abstract Factory__ pattern.
- The goal of the __Adapter__ pattern is to make classes work together. While Bridge aims to generalize abstractions and implementations.

## Composite - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Composite)
### When
- Objects can be decomposed to tree structures.
- Objects and sub objects should be treated in a similar fashion by the client. 
### Effects
- Objects mimics structured hierarchies of its business representation.
- Simplifies the client by omitting type checking since composites are treated uniformly.
- Design can become too generic, filtering and restricting composites fallback to type checking.
### Relationships
- Composites can be traversed using the __Iterator__ pattern.
- Operations shared in the composite structure can be abstracted by the __Visitor__ pattern.
- The links in the __Chain of Responsibility__ and __Decorator__ pattern is usually a composite.

## Decorator - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Decorator)
### When
- Functionality must be added dynamically -
- And removed dynamically.
- Adding responsibilites to classes produces unecessary amount of subclasses.
### Effects
- More flexible and concise addition of responsibility to objects.
- Features are added when they are needed. Classes are not prepared for the future uses making it lightweight.
- Although the decorator is transparent to the client, type checking can see it different from the what it is decorating on.
- Loss of cohesion since the responsibility is spread over little objects.
### Relationships
- The __Adapter__ pattern combines functionality while the decorator pattern adds it.
- A decorator is a __Composite__ with one node.
- The decorator pattern changes the surface of a class, while the __Strategy__ pattern changes the insides of it.

## Facade - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Facade)
### When
- A complex system can be exposed into a simple abstraction.
- Responsibility of the client on implementing abstraction of a system can be hidden.
- A system can be layered.
### Effects
- Hides the complexity of a system, making it easier to implement.
- Only hides the complex system, client still has a choice to directly use the complex system.
- Reduces clients coupling to the abstraction implmentation logic.
### Relationships
- __Abtract Factory__ can be used with facade to centralize object creation.
- One instace of Facade is usually enough allowing it to be __Singleton__.
- The __Mediator__ pattern is similar in a way to Facade in that it centralizes the communication logic of components. The components of the Mediator are aware of the Mediator while the Facade is trasparent to the systems it encapsulate.

## Flyweight - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Flyweight)
### When
- A large amounts of unchanging objects are used within the application.
- The storage performance suffers from the said objects.
- The state (extrinsic) or operation of the objects can be abstracted outside of it.
- Abtracting the state (extrinsic) allows objects to be shared.
- Absence of type checking of the said objects.
### Effects
- Reduced space consumption.
### Relationships
- The __Composite__ pattern allows optimized querying of the Flyweight objects by representing them in a tree structure.
- Large amounts of __State__ and __Strategy__ objects can be represented as Flyweights.

## Proxy - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Proxy)
### When
- Object creation can be deferred for responsiveness.
- Access restrictions must be imposed to objects.
- Objects being used must be tracked for caching.
### Effects
- Object accessing and creation is optimized.
- A layer of safety is added to the object being proxied.
### Relationships
- The proxy pattern rivals the __Adapter__ pattern. The proxy pattern copies the subject (class) interface while the __Adapter__ pattern created a new interface to connect subjects.
- The proxy patter also rivals the __Decorator__ pattern. The proxy pattern controls access to the subject while the __Decorator__ pattern adds functionalities to the subjects.