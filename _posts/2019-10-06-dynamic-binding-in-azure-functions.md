---
layout: post
title: Dynamic Binding in Azure Functions
date: 2019-10-06 22:48:20 +0800
description: Tutorial on how to implement dynamic binding in Azure Functions
tags: [Azure, Serverless, C#]
---

Binding Attributes in a C# flavored Azure Function offers great functionality out of the box. Most of the time these declarative bindings are enough. The deal of having a declarative implementation arises when you want more control on the binding or when and how it is going to be triggered. Digging under the hood, these Binding Attributes can be replaced with `IBinder` or `Binder` instance which allows for imperative dynamic binding.

# The Declarative Catch (Attribute-based Binding)
// Code

The code above contains two (2) Blob Bindings, both directions are set to output. The Function is also triggered with a Blob input. The first binding will serve as the output destination when the input Blob is successfully processed and the second binding will serve as the output destination when the processing fails on the input Blob.

- Uploading the Blobs – __Passing__
// Image
- Uploading the Blobs – __Failing__
// Image

From both tests we can confirm that both transfers the Blob in the correct directory, but an empty Blob is created on Failing directory for the Passing Test and vice versa. The culprit is the declarative binding which __happens at compile-time__, even if the supposed output is directed at one of the Streams.

# The Imperative Solution (Dynamic Binding)
The solution to the dilemma is to __move the binding execution from compile-time to run-time__. It can be achieved by “Injecting” the `IBinder` or `Binder` instance to the Function.
```csharp
public interface IBinder
{
    Task<T> BindAsync<T>(
        Attribute attribute, 
        CancellationToken cancellationToken = default(CancellationToken));
}
```
The interface provides one (1) method called BindAsync which accepts an Attribute parameter.

1. “Inject” the `IBinder` or `Binder` instance to the Function's parameter. 
The binding parameters can now be replaced with a single `IBinder` or `Binder` instance.
// Code

2. Create the Attribute Object
Instead of declaring the arguments in the parameter attribute. An `Attribute` object must now be created, it can also be an `Attribute[]`. For the Blob Binding, a `BlobAttribute` must be initialized with the Path and Direction. Optionally, the `StorageAccountAttribute` can be set. 
// Code

The following Attributes for common binding are:
- [Storage Account](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)
- [Blob](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs)
- [Queue](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Queues/QueueAttribute.cs)
- [Table](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Tables/TableAttribute.cs)

_Note: Other Attributes can be found in Microsoft.Azure.WebJobs.Extensions.{binding}. Refer to the list of supported binding [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings#supported-bindings)._

3. Invoke the `BindAsync` method
Pass the Attributes and set the Type. The Type is specialized to the binding, this is usually the type on the parameter when an Attribute-based Binding is used. The Blob Binding prefers a `Stream` or `CloudBlockBlob` type for better memory utilization. 
// Code
_Note: From the Microsoft Docs, the Binder instance must be properly disposed by wrapping in a `Using` Block. In C# 8, the `Using` Blocks can now be an Expression for brevity._

# Implementation
// Code
Upon triggering the Function again, the tests showed that the lingering empty Blobs does not get created anymore. Since __the binds are dynamically made on run-time.__
- Uploading the Blobs __(Passing)__
// Image
- Uploading the Blobs __(Failing)__
// Image

Dynamic Binding offers flexibility and control on Azure Functions Bindings as well as minimizing parameter clutter. A lot more applications can take advantage of this technique versus the traditional Attribute-based Binding where conditional operations must be performed on the Bindings.  
