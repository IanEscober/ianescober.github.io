---
title: Design Pattern Secrets - Structural
date: Y2020-02-09 21:47:20 +0800
categories: [Tech, C#]
tags: [C#, Design Patterns, Series]
---

Continuing from the last post covering the [Creational Design Pattern Secrets](https://ianescober.github.io/posts/design-patterns-secrets-creational/), this post will also be structured again in:
- __When__
- __Effects__
- __Relationships__

As said before this will be a quick catalog of information that are rarely highlighted about Design Patterns. Examples can be found at my [Github](http://github.com/ianescober/designpatterns).

# Creational Patterns
## Adapter - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Adapter)
### When
- An interface of a class do not coincide with your needs.
- A common class that mediates other incompatible classes with each other.
- A need to use several classes or interfaces without inheriting.
### Effects
- A single reusable class can work with many other classes.
- The Adapter class can override some behavior of the classes it is supporting.
- The Adapter class is prone to "doing to much".

### Relationships
- Similar to __Bridge__ which is about seperating interfaces from implementation while Adapter is about modifying interfaces to make it work with other.
- Similar to __Decorator__ which adds funcionality to classes without altering its interface. Supporting recursive implementation, which is not possible with Adapter.
- Similar to __Proxy__ which creates a "proxy" to make classes work with each other.

## Bridge - [example](https://github.com/IanEscober/DesignPatterns/tree/master/src/Bridge)
### When
- A need to prevent coupling of abstraction and implementation.
- Abstractions of implementations can be mixed and match or vice versa.
- Changes to abstractions should be tranparent to the clients.
### Effects
- Assignment of implementation to abstraction can happen dynamically.
- Implementations and abstractions can be scaled independently.
- Clients are unaffected by changes in implementations and abstractions.
### Relationships
- A Bridge can be create by __Abstract Factory__.
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