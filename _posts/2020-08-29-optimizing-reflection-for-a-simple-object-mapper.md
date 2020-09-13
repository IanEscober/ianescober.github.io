---
title: Optimizing Reflection for a Simple Object Mapper
date: Y2020-08-29 19:45:20 +0800
categories: [Tech, Csharp]
tags: [Csharp, Reflection, ORM, Database]
seo:
  date_modified: 2020-08-29 19:45:20 +0800
---

![Mapping](https://drive.google.com/uc?export=view&id=1-zsb1XlkT4kJhnmouLqopRWZTV2d50Dr)

Currently in a project that I am in uses [Entity Framework](https://docs.microsoft.com/en-us/ef/) to do - well all database actions. And one of which is was mapping database data to objects. All was well until the growing scale of the project was affected by the performance of EF. The projected "almost entirely" migrated away from EF, the queries are now hand-made which showed great performance improvements. Performant query generation was one of the biggest performance hits of EF. Which is why explicitly creating queries have far better perfromance. But EF is still being used in the project not for database actions anymore but for mapping database data to objects. Object mapping was not much of a perfomance hit compared to the latter so it was not much of an issue. But the results of the project's queries are always flat and EF "object mapping magic" was not utilized anymore. The overhead of EF is quite significant if object mapping was it's only job. Lo and behold an idea was born. What if we make an object mapper!

# Setting the Baseline
Surveying other projects in the company. One caught my attention, it uses [Json.NET](https://www.newtonsoft.com/) to map database data to objects. The process starts from storing the column names as the `key` of a `Dictionary` and the row values as the `value`. The `Dictionary` is then serialized to JSON, then from JSON it is deserialized to an object. Pretty clever right? Performance was also incredible (see later at benchmarks) despite the work on the back and forth serialization and deserialization of the object.

```csharp
public static T Map<T>(this DataReader reader) 
{
    var objectDictionary = new List<Dictionary<string, object>>();
    var columnNames = new List<string>();

    // Store Column Names
    for (var i = 0; i < reader.FieldCount; i++)
    {
        columnNames.Add(reader.GetName(i));
    }

    // Iterate and store column name and row value to a Dictionary
    foreach(var columnName in columnNames)
    {
        objectDictionary.Add(new Dictionary<string, object> { { columnName, reader[columnName] } });
    }

    // Serialize then deserialize
    var json = JsonConvert.SerializeObject(objectDictionary);
    var result = JsonConvert.DeserializeObject<T>(json);

    return result;
}
```

This could work well but it has a dependency on an external library (Json.NET). Our goal is to create an object mapper with only the BCL's (Base Class Library). For that our only option is to use - reflection.

# Looking at Reflection
Reflection allows us to peek at the properties of an object, specifically its name, getter, and setter. With the column names given to us by the `DataReader`, we can match it with the object properties using reflection and set the row value to the property using its setter.

```csharp
public static T Map<T>(this DataReader reader) where T : new()
{
    var objectContainer = new T();
    var objectType = typeof(T);
    var properties = objectType.GetProperties(BindingFlags.Public | BindingFlags.Instance);

    for (int i = 0; i < reader.FieldCount; i++)
    {
        var columnName = reader.GetName(i);
        var propertyInfo = properties.SingleOrDefault(prop => prop.Name == columnName);

        if (propertyInfo != null)
        {
            var value = reader[columnName];
            propertyInfo.SetValue(objectContainer, value);
        }
    }

    return objectContainer;
}
```

A lot of reflection is going on here. Let's break it down.

1. Create an object container to be hydrated. - _Line 3_
   - A generic constraint of `new()` allows us to do `new T()`, it specifies that the type must have a parameterless constructor. 
2. Get the type of the object. - _Line 4_
3. Get the properties of the object type. - _Line 5_
   - `BindingFlags.Public | BindingFlags.Instance` ensure that we only read the public and non static properties of the object.
4. Check if the column name matches to any properties in the object type. - _Line 10_
5. Use the matched property based on the column name and use its setter to assign the value to the container object. - _Line 15_

We manage to remove the dependecy from Json.NET by only using the BCL's. Now let's benchmark it against the Json.NET implementation.

|-----------------|------------|-----------------|
| Mapper          | Rows       | Mean            |
|-----------------|------------|-----------------|
| Reflection      | 1          | 868.8 µs        |
| Json            | 1          | 692.2 µs        |
| Reflection      | 10         | 8033.6 µs       |
| Json            | 10         | 2593.4 µs       |
| Reflection      | 100        | 78,895.3 µs     |
| Json            | 100        | 21,849.9 µs     |
| Reflection      | 10000      | 7,798,628.1 µs  |
| Json            | 10000      | 2,106,959.9 µs  |

__Single Row__ - The performance of both implementations have negligible difference.

__Multiple Rows__ - A noticeable difference can be seen. The mapper using reflection is about 4x slower than the one using Json.NET. The trend continues as the number of rows increases.

# Why is it slow?
.NET reflection is notoriously [slow inside loops](https://stackoverflow.com/questions/25458/how-costly-is-net-reflection). Our culprit is just one line of code. This line of code looks harmless at first but it is the reason why our performance tanks. __Each time `SetValue` is called it iterates to the object to find the property which it will set its value.__

```csharp
propertyInfo.SetValue(objectContainer, value);
```

The immediate idea is to cache it to prevent iteration. We have this line of code that gets the properties of the object. At first glance it looks like it "caches the properties", why is it not providing any performance gains? Because it only caches the "object type" properties.

```csharp
var properties = objectType.GetProperties(BindingFlags.Public | BindingFlags.Instance);
```

To solve this property lookup on each loop we have to dig deeper in reflection and solutions are rather unusual once we dive into reflection. Let's see why.

# Optimizing Reflection
The question is _"how to cache the object property to prevent looking it up every operation?"_. We are used to caching data but in our situation there is no data, because we are just setting the data. What we have is an "operation" of setting the data to the property of the object. What if we cached that "action" instead. The idea was inspired by Jon Skeet's blog post - [Making Reflection Fly and Exploring Delegates](https://codeblog.jonskeet.uk/2008/08/09/making-reflection-fly-and-exploring-delegates/). The blog post explores into great detail about reflection and optimizing it. The solution is quiet complex but we can distill it into the code below using modern C# types. 

```csharp
public static Action<T, object> CacheSetter<T>(MethodInfo method)
{
    return (model, value) => method.Invoke(model, new object[] { value });
}
```

1. The method accepts a method info, which is the property's setter method. 
   - `MethodInfo method`
   - `public name { set; }`
2. It returns an action delegate with the same signature as `propertyInfo.SetValue`. 
   - `Action<T, object>`
   - `(model, value) => ...`
3. The body of the action invokes the property setter and passing the model and value.
   - `... => method.Invoke(model, new object[] { value })`

Using the method, we can now _"cache the object property setters"_ and store it to a dictionary that we can look up each iteration on the object.

```csharp
public static Dictionary<string, Action<T, object>> CacheProperties<T>(IDataReader reader) where T : new()
{
    var properties = new Dictionary<string, Action<T, object>>();

    for (int i = 0; i < reader.FieldCount; i++)
    {
        var fieldName = reader.GetName(i);
        var methodInfo = typeof(T).GetProperty(reader.GetName(i))?.GetSetMethod();

        if (methodInfo != null)
        {
            var setter = CacheSetter<T>(methodInfo); // Cache the property setter
            properties.Add(fieldName, setter);
        }
    }

    return properties;
}
```

The updated reflection mapper utilizing the property caching can be seen below.

```csharp
public static T Map<T>(IDataReader reader) where T : new()
{
    var result = new T();
    var properties = CacheProperties<T>(reader);
    
    foreach (var property in properties)
    {
        var value = reader[property.Key];
        property.Value(result, value);
    }

    return result;
}
```

# The Benchmark
Now lets test the optimized reflection mapper against the Json and the old implementation.

|-----------------------|------------|-----------------|
| Mapper                | Rows       | Mean            |
|-----------------------|------------|-----------------|
| Reflection            | 1          | 868.8 µs        |
| Optimized Reflection  | 1          | 933.8 µs        |
| Json                  | 1          | 692.2 µs        |
| Reflection            | 10         | 8033.6 µs       |
| Optimized Reflection  | 10         | 2837.4 µs       |
| Json                  | 10         | 2593.4 µs       |
| Reflection            | 100        | 78,895.3 µs     |
| Optimized Reflection  | 100        | 21,017.3 µs     |
| Json                  | 100        | 21,849.9 µs     |
| Reflection            | 10000      | 7,798,628.1 µs  |
| Optimized Reflection  | 10000      | 2,026,914.6 µs  |
| Json                  | 10000      | 2,106,959.9 µs  |

We achieved impressive perfromance gains to the point that it rivals the Json implementation. We've increased the performance of reflection by 4x. Starting at 100 rows, the optimized reflection mapper actually pulls ahead of the Json implementation but it's not that significant of about -3%.

The full benchmark can be seen below. A third party library was used for comparison ([FastMember](https://github.com/mgravell/fast-member)). It's the one used by [Dapper](https://github.com/StackExchange/Dapper) (a micro-ORM) for reflection purposes.

![Benchmark Results](https://drive.google.com/uc?export=view&id=1mcaX2V2KQLX1vmTtEC3JvxbALkZ5EK4m)

- [Bechmark Code](https://github.com/IanEscober/ObjectMapperBenchmark)

# Conclusion
Reflection is incredibly powerful but one must have the knowledge of its performance penalties in order to make an efficient solution out of it. In this post we saw that by caching the "action" of the property setters greatly improved the performance of reflection on object assignment. Pretty radical solution to the world that is used to caching data.

Even though we have not achieved a significant performance over of the Json implementation, the optimized reflection was able to match it at more significant workloads. And we expect this trend to continue as more rows are mapped.

# Further Optimization
This is only the start of optimizing reflection. Several optimization techniques can still be applied to our reflection object mapper. For instance, instructing the compiler to what to do - reverse engineering .NET. But that is a story for another day.

For further reading you can look at:

- [Optimizing Object Instantiation](https://devblogs.microsoft.com/premier-developer/dissecting-the-new-constraint-in-c-a-perfect-example-of-a-leaky-abstraction/)
- [IL OpCodes](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.emit.opcodes?view=netcore-3.1)
- [Roslyn Scripting](https://docs.microsoft.com/en-us/archive/blogs/csharpfaq/introduction-to-the-roslyn-scripting-api)