---
title: Optimizing Reflection for a Simple Object Mapper
date: Y2020-08-29 19:45:20 +0800
categories: [Tech, Csharp]
tags: [Csharp, Reflection, ORM, Database]
---

Currently in a project that I am in uses [Entity Framework](https://docs.microsoft.com/en-us/ef/) to do - well all database actions. And one of which is - which EF was - mapping database data to objects. All was well until the growing scale of the project was affected by the performance of EF. The projected "almost entirely" migrated away from EF, the queries are now hand-made which showed great performance improvements. Performant query generation was one of the biggest performance hits of EF. Which is why explicitly creating queries have far better perfromance. But EF is still being used in the project not for database actions anymore but for mapping database data to objects. Object mapping was not much of a perfomance hit compared to the latter so it was sufficient. But the results of the project's queries are always flat and EF "object mapping magic" was not utilized anymore. Also the overhead of EF is quite significant if object mapping was it's only job. Lo and behold an idea was born. What if we make an object mapper!

# Setting the Baseline
I started the journey by surveying other projects in the company I'm in. One caught my attention, it uses [Json.NET](https://www.newtonsoft.com/) to map database data to objects. The process starts from storing the column names as the `key` of `Dictionary` and the row values as the `value`. The `Dictionary` is then serialized to JSON, then from JSON it is deserialized to an object. Pretty clever right? Performance was also incredible (see later at benchmarks).

// Code

This could work well but it has a dependency on an external library (Json.NET). Our goal is to create an object mapper with only the BCL's (Base Class Library). For that our only option is to use - reflection.

# Looking at Reflection
Reflection allows us to peek at the properties of an object, specifically its name, getter, and setter. With the column names given to us by the `Data Reader`, we can match it with the object properties to be hydrated using reflection. Also we can set the row value to the propert using its setter through reflection.

// Code
