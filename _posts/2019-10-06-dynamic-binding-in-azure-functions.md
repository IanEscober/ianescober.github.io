---
title: Dynamic Binding in Azure Functions
date: 2019-10-06 22:48:20 +0800
categories: [Tech, Azure]
tags: [Azure, Serverless, Functions, Blob]
seo:
  date_modified: 2019-12-24 14:15:57 +0800
---

![Azure Functions Binding](https://drive.google.com/uc?export=view&id=1Ba3ZR6SyMFvhSZ2ubmAghuyOH_KxIDrp)
Binding Attributes in a C# flavored Azure Function offers great functionality out of the box. Most of the time these declarative bindings are enough. The deal of having a declarative implementation arises when you want more control on the binding or when and how it is going to be triggered. Digging under the hood, these Binding Attributes can be replaced with `IBinder` or `Binder` instance which allows for imperative dynamic binding.

# The Declarative Catch (Attribute-based Binding)
```csharp
[FunctionName("Bindings")]
[StorageAccount("AzureWebJobsStorage")]
public static async Task Run(
    [BlobTrigger("container/{blobName}.{ext}")] Stream input,
    [Blob("pass/{blobName}.{ext}", FileAccess.Write)] Stream outputPass,
    [Blob("fail/{blobName}.{ext}", FileAccess.Write)] Stream outputFail,
    string blobName,
    string ext)
{ ... }
```

The code above contains two (2) Blob Bindings, both directions are set to output (e.g. `FileAccess.Write`). The Function is also triggered with a Blob input. The first binding will serve as the output destination when the input Blob is successfully processed and the second binding will serve as the output destination when the processing fails on the input Blob.

- Uploading a __Passing__ blob
![Attribute Binding Result](https://drive.google.com/uc?export=view&id=1X69Aad1RyPVM5TrPT7Gz2pflR0uuPiLI)
![Attribute Binding Pass](https://drive.google.com/uc?export=view&id=1EJI2d1gZAFcMr0QdcqNmGJ0DKTypEu-8)
![Attribute Binding Fail](https://drive.google.com/uc?export=view&id=1xHUU371Bm_NQdh7fOcroMHNV3Zt7zpIX)

From both tests we can confirm that both transfers the Blob in the correct directory, but an empty Blob is created on Failing directory for the Passing Test and vice versa. The culprit is the declarative binding which __happens at compile-time__, even if the supposed output is directed at one of the Streams.

# The Imperative Solution (Dynamic Binding)
The solution to the dilemma is to __move the binding execution from compile-time to run-time__. It can be achieved by “Injecting” the `IBinder` or `Binder` instance to the Function. The interface provides one (1) method called BindAsync which accepts an Attribute parameter.
```csharp
public interface IBinder
{
    Task<T> BindAsync<T>(
        Attribute attribute, 
        CancellationToken cancellationToken = default(CancellationToken));
}
```

1. “Inject” the `IBinder` or `Binder` instance to the Function's parameter. 
The binding parameters can now be replaced with a single `IBinder` or `Binder` instance.
```csharp
[FunctionName("Bindings")]
[StorageAccount("AzureWebJobsStorage")]
public static async Task Run(
    [BlobTrigger("container/{blobName}.{ext}")] Stream input,
    IBinder binder,
    string blobName,
    string ext)
{ ... }
```

2. Create the Attribute Object
Instead of declaring the arguments in the parameter attribute. An `Attribute` object must now be created, it can also be an `Attribute[]`. For the Blob Binding, a `BlobAttribute` must be initialized with the Path and Direction. Optionally, the `StorageAccountAttribute` can be set if it is part of the binding option (e.g. `Connection = "AzureWebJobsStorage"`). 
```csharp
var attributes = new Attribute[]
{
     new BlobAttribute(path, FileAccess.Write),
    // new StorageAccountAttribute(FunctionConfig.Blob)
};
```
The following Attributes for common binding are:
- [Storage Account](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)
- [Blob](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs)
- [Queue](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Queues/QueueAttribute.cs)
- [Table](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Tables/TableAttribute.cs)
- [Other Attributes](https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings#supported-bindings) can be found in _Microsoft.Azure.WebJobs.Extensions.{binding}_

3. Invoke the `BindAsync` method
Pass the Attributes and set the Type. The Type is specialized to the binding, this is usually the type on the parameter when an Attribute-based Binding is used. The Blob Binding prefers a `Stream` or `CloudBlockBlob` type for better memory utilization.
```csharp
using (var output = await binder.BindAsync<Stream>(attributes))
{
    // Transfer logic
}
```
_Note: From the Microsoft Docs, the Binder instance must be properly disposed by wrapping in a `Using` Block. In C# 8, the `Using` Blocks can now be an Expression for brevity._

# Implementation
```csharp
[FunctionName("Bindings")]
public static async Task Run(
    [BlobTrigger("container/{blobName}.{ext}")] Stream input,
    IBinder binder,
    string blobName,
    string ext)
{
    var path = string.Empty;
    var storageAccount = "AzureWebJobsStorage";

    try
    {
        // Process input blob
        path = $"pass/{blobName}.{ext}";
    }
    catch(Exception ex)
    {
        path = $"fail/{blobName}.{ext}";
    }

    var attributes = SetAttributes(path, storageAccount);
    await TransferBlobAsync(input, attributes);
}

private static Attribute[] SetAttribute(string path, string storageAccount)
{
    var attributes = new Attribute[]
    {
        new BlobAttribute(path, FileAccess.Write),
        new StorageAccountAttribute(storageAccount)
    };
}

private static async Task TransferBlobAsync(Stream input, Attribute[] attributes)
{
    using (var output = await binder.BindAsync<Stream>(attributes))
    {
        // Transfer logic
    }
}
```
Upon triggering the Function again, the tests showed that the lingering empty Blobs does not get created anymore. Since __the binds are dynamically made on run-time.__

- Uploading a __Passing__ blob
![Dynamic Binding Result](https://drive.google.com/uc?export=view&id=1RfBhgVpIdbZV3PIprugpeeh-O2xH8Uiy)
![Dyanmic Binding Pass](https://drive.google.com/uc?export=view&id=1EJI2d1gZAFcMr0QdcqNmGJ0DKTypEu-8)
![Dynamic Binding Fail](https://drive.google.com/uc?export=view&id=1CfCvX1h1aW1i-NzDKJuu6mM-JyLsmzjp)

Dynamic Binding offers flexibility and control on Azure Functions Bindings as well as minimizing parameter clutter. A lot more applications can take advantage of this technique versus the traditional Attribute-based Binding where conditional operations must be performed on the Bindings.  
