---
title: Creating a simple Kubernetes operator in .NET (C#)
description: Creating a Kubernetes operator can seem a bit overwhelming at first. To help, there’s a simple NuGet package called CanSupportMe.Operator which can be as simple as watching for a secret or config map or creating a custom resource definition (CRD) and watching that. You then get call backs for new items, modified items, reconciling* items, deleting* items and deleted items. (* CRDs only)
date: 2024-02-08
categories:
  - Kubernetes
  - .NET
  - C\#
  - Tutorials
---
# Creating a simple Kubernetes operator in .NET (C#)

***Note: I'd recommend building a Kubernetes operator using Go as it is the language of choice for Kubernetes. This article is more of a proof of concept.***

Creating a Kubernetes operator can seem a bit overwhelming at first. To help, there’s a simple NuGet package called [CanSupportMe.Operator](https://www.nuget.org/packages/CanSupportMe.Operator/) which can be as simple as watching for a secret or config map or creating a custom resource definition (CRD) and watching that. You then get call backs for new items, modified items, reconciling* items, deleting* items and deleted items. (* CRDs only)

The call backs also expose a data object manager which lets you do things like create secrets and config maps, force the reconciliation of an object, clear all finalizers and check if a resource exists.

## Example

The follow is a console application with the following packages installed:

```powershell
dotnet add package Microsoft.Extensions.Configuration.Abstractions
dotnet add package Microsoft.Extensions.Hosting
dotnet add package Microsoft.Extensions.Hosting.Abstractions
dotnet add package CanSupportMe.Operator
```

If you’re not use a context with a token in your KUBECONFIG, use the commented out failover token lines to specify one (how to generate one is on the NuGet package page).

```csharp
using CanSupportMe.Operator.Extensions;
using CanSupportMe.Operator.Models;
using CanSupportMe.Operator.Options;
using Microsoft.Extensions.Hosting;

// const string FAILOVER_TOKEN = "<YOUR_TOKEN_GOES_HERE>";

try
{  
  Console.WriteLine("Application starting up");

  IHost host = Host.CreateDefaultBuilder(args)
    .ConfigureServices((context, services) =>
    {
      services.AddOperator(options =>
      {
        options.Group = "";
        options.Kind = "Secret";
        options.Version = "v1";
        options.Plural = "secrets";
        options.Scope = ResourceScope.Namespaced;
        options.LabelFilters.Add("app.kubernetes.io/managed-by", "DemoOperator");

        options.OnAdded = (kind, name, @namespace, item, dataObjectManager) =>
        {
          var typedItem = (KubernetesSecret)item;

          Console.WriteLine($"On {kind} Add: {name} to {@namespace} which is of type {typedItem.Type} with {typedItem.Data?.Count} item(s)");
        };

        options.FailoverToken = FAILOVER_TOKEN;
      });
    })
    .Build();

  host.Run();
}
catch (Exception ex)
{
  Console.WriteLine($"Application start failed because {ex.Message}");
}
```

To create a secret that will trigger the `OnAdded` call back, create a file called `secret.yaml` with the following contents:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret-with-label
  namespace: default
  labels:
    app.kubernetes.io/managed-by: DemoOperator
stringData:
  notImportant: SomeValue
type: Opaque
```

Then apply it to your Kubernetes cluster using the following command:

```bash
kubectl apply -f secret.yaml
```

This should result in the following being output to the console:

```
On Secret Add: test-secret-with-label to default which is of type Opaque with 1 item(s)
```