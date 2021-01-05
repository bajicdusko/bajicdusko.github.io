---
layout: "post"
title: "Registering all types as generic interfaces in assembly in dotnet core"
date: "2021-01-05 13:00"
categories: [tips, dotnet]
tags: [dotnet, core, dependency injection, generic, interface]
description: "We will show how to register all types implementing generic interface, 
to resolve as generic interface in dotnet core"
---

Let's say that we have an interface in your app as `IRepository<T>` which has dozens of implementations.
Wouldn't it be so nice to define registration rule for dependency injection container, only once 
so that it can cover all existing, and future repositories, without additional effort.  

This is something we want to avoid:

```
serviceCollection.AddTransient(typeof(IRepository<User>), typeof(UserRepository));
serviceCollection.AddTransient(typeof(IRepository<Company>), typeof(CompanyRepository));
serviceCollection.AddTransient(typeof(IRepository<Car>), typeof(CarRepository));
```

If you've used Autofac before, something like this could be defined in a couple of lines:

```csharp
public static void UseRepositories(this ContainerBuilder builder) 
{
    builder.RegisterAssemblyTypes(Assembly.GetExecutingAssembly())
        .AsClosedTypesOf(typeof(IRepository<>))
        .AsImplementedInterfaces();
}
```
Basically you would get current assembly, select all types implementing `IRepository<>` and register
them as interface they implement, which is at least `IRepository<>`.  
After this, we're able to inject `UserRepository` as `IRepository<User>`.

In dotnet core, you wont use Autofac most probably. We still don't have those handy methods like `AsClosedTypesOf` or `AsImplementedInterfaces`
therefore, we'd have to implement it on our own, something like this:


```csharp
Assembly.GetExecutingAssembly()
    .GetTypes()
    .Where(a => a.Name.EndsWith("Repository") && !a.IsAbstract && !a.IsInterface)
    .Select(a => new { assignedType = a, serviceTypes = a.GetInterfaces().ToList() })
    .ToList()
    .ForEach(typesToRegister =>
    {
        typesToRegister.serviceTypes.ForEach(typeToRegister => services.AddScoped(typeToRegister, typesToRegister.assignedType));
    });
```

What this block of code do actually?

First we get all types from the assembly:

```csharp
...
Assembly.GetExecutingAssembly()
    .GetTypes()
...
```

then we filter those types to the repository classes

```csharp
...
.Where(a => a.Name.EndsWith("Repository") && !a.IsAbstract && !a.IsInterface)
...
```

and we define pairs of types with the list of interfaces they implement

```csharp
...
.Select(a => new { assignedType = a, serviceTypes = a.GetInterfaces().ToList() })
    .ToList()
...
```

at the end we just itterate through the pairs of types and register each of them as the interface they implement

```csharp
...
.ForEach(typesToRegister =>
    {
        typesToRegister.serviceTypes.ForEach(typeToRegister => services.AddScoped(typeToRegister, typesToRegister.assignedType));
    });
...
```

Now we're able to inject `UserRepository` as `IRepository<User>`, without the need for explicit registration in DI container of each type.
