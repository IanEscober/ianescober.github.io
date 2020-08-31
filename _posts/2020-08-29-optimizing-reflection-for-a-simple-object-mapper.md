---
title: Optimizing Reflection for a Simple Object Mapper
date: Y2020-08-29 19:45:20 +0800
categories: [Tech, Csharp]
tags: [Csharp, Reflection, ORM, Database]
---

Currently in a project that I am in uses [Entity Framework](https://docs.microsoft.com/en-us/ef/) to do - well all database actions. And one of which is - which EF was - mapping database data to objects. All was well until the growing scale of the project was affected by the performance of EF. The projected "almost entirely" migrated away from EF, the queries are now hand-made which showed great performance improvements. Performant query generation was one of the biggest performance hits of EF. Which is why explicitly creating queries have far better perfromance. But EF is still being used in the project not for database actions anymore but for mapping database data to objects. Object mapping was not much of a perfomance hit compared to the latter so it was sufficient. But the results of the project's queries are always flat and EF "object mapping magic" was not utilized anymore. Also the overhead of EF is quite significant if object mapping was it's only job. Lo and behold an idea was born. What if we make an object mapper!

# Setting the Baseline
Surveying other projects in the company. One caught my attention, it uses [Json.NET](https://www.newtonsoft.com/) to map database data to objects. The process starts from storing the column names as the `key` of a `Dictionary` and the row values as the `value`. The `Dictionary` is then serialized to JSON, then from JSON it is deserialized to an object. Pretty clever right? Performance was also incredible (see later at benchmarks) despite the work on serializatio and deserialization.

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

    // Iterate and store column name and row value to the Dictionary
    while (reader.Read())
    {
        foreach(var columnName in columnNames)
        {
            objectDictionary.Add(new Dictionary<string, object> { { columnName, reader[columnName] } });
        }
    }

    // Serialize then deserialize
    var json = JsonConvert.SerializeObject(objectDictionary);
    var result = JsonConvert.DeserializeObject<T>(json);

    return result;
}
```

This could work well but it has a dependency on an external library (Json.NET). Our goal is to create an object mapper with only the BCL's (Base Class Library). For that our only option is to use - reflection.

# Looking at Reflection
Reflection allows us to peek at the properties of an object, specifically its name, getter, and setter. With the column names given to us by the `DataReader`, we can match it with the object properties using reflection and set the row value to the property using its setter through reflection also.

```csharp
public static T Map<T>(this DataReader reader) where T : new()
{
    var result = new List<T>();
    var objectContainer = new T();
    var objectType = typeof(T);
    var properties = objectType.GetProperties(BindingFlags.Public | BindingFlags.Instance);

    while (reader.Read())
    {
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

        result.Add(result);
    }
}
```

A lot of reflection is going on here. Let's break it down.

1. Create an object container to be hydrated. - Line 4
2. Get the type of the object. - Line 5
3. Get the properties of the object type. - Line 6
   - `BindingFlags.Public | BindingFlags.Instance` ensure that we only read the public and non static properties of the object.
4. Check if the column name matches any properties in the object type. - Line 13
5. Use the matched property based on the column name and use its setter to assign the value to the container object. - Line 18
