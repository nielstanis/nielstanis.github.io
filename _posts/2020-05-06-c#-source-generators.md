---
layout: post
title: C# Source Generators
categories: [.NET 5, C#]
tags: [compiler, preview]
---

A week ago Microsoft released a preview of a new compiler feature called [C# Source Generators](https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/). This feature allows to inspect existing source-code and generate additional C# source files that will be added to the compiled output. One of the goals is to use this in order to remove the need for any (slower) dynamic runtime behavior that rely on reflection. I also think there are benefits from security perspective that can be added to that list, let's look into that a bit further. 

# Source Generator

When a Source Generator gets added to a compile pipeline it will get executed when the application gets build. It has the ability to inspect the programs source artifacts and create additional files that will be added to the output. If you look into the [design document](https://github.com/dotnet/roslyn/blob/master/docs/features/source-generators.md) the concept is also called _compile time metaprogramming_ Source Generators have a strong relationship with existing Roslyn Analyzers that allow you to inspect both syntax and the semantic models of the compilation. 

# Application Parts in ASP.NET Core MVC

When you create an ASP.NET Core MVC application and add new functionality with the help of either Controllers and Views or Razor Pages and you start the app it all just _works_. One key component in all of this is the [ApplicationPartsManager](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.applicationparts.applicationpartmanager) that will contain a list of [FeatureProviders](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.applicationparts.iapplicationfeatureprovider) and those will populate the needed content for e.g. Razor Pages, Controllers, TagHelpers and Views. Read the [Share controllers, views, Razor Pages and more with Application Parts](https://docs.microsoft.com/en-us/aspnet/core/mvc/advanced/app-parts) article in the Microsoft documentation for more details on how it works. 

# What is exposed by ASP.NET Core MVC?

The whole way of _automagically_ resolving the content can be really helpful when developing one big downside is the fact that there is a risk of exposing more than intended. From the main MVC project, the one that constructs the host builder, any referenced project or added package that also reference the [Microsoft.AspNetCore.Mvc](https://www.nuget.org/packages/microsoft.aspnetcore.mvc) package will automatically be scanned for classes that either end with `...Controller` (including inheritance) or are annotated with `Microsoft.AspNetCore.Mvc.ControllerAttribute`. Each identified controller will the have _all_ it's public methods exposed and keep in mind that _any_ used third-party package has the ability to do that. It is not unseen that packages get compromised (there are a lot of ways!) by people who have got bad intentions. Of course it's possible to determine a list of exposed controllers at run time but I think it's better to have more control (e.g. whitelist of allowed ones) from the start.

# Determine exposed controllers at compile time

Exactly a year ago David Fowler shared a [tweet](https://twitter.com/davidfowl/status/1123777569621409793) in which he explained that one of the design goals of ASP.NET Core was to remove the scanning of assemblies at start up to find _things_ that are exposed. In the introduction blog-post of C# Source Generators you can read that this is exactly what the new compiler feature will bring to ASP.NET Core MVC. Currently the demo's show some basic scenario's on how it can be used, I'm really looking forward on seeing an implementation of it in the ASP.NET Core MVC packages. From static analysis perspective the benefit of doing this will result in less need to understand whats happening at runtime with the dynamically resolving of controllers because it's all just part of the compiled assembly.
