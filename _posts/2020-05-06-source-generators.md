---
layout: post
title: C# Source Generators
categories: [.NET5, CSharp]
tags: [compiler, preview]
---

A week ago Microsoft released a preview of a new compiler feature called [C# Source Generators](https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/){:target="_blank"}. This feature allows to inspect existing source-code and generate additional C# source files that will be added to the compiled output. One of the goals is to use this in order to remove the need for any (slower) dynamic runtime behavior that rely on reflection. There are also benefits from security perspective that can be added to that list, let's look into that a bit further! 

# Source Generator

When a Source Generator is added to a .NET Core project it will be executed when the project is being build. It has the ability to inspect the programs source artifacts and create additional files that will be added to the output. The concept is also called _Compile Time Metaprogramming_ as explained in the [design document](https://github.com/dotnet/roslyn/blob/master/docs/features/source-generators.md){:target="_blank"}. Source Generators have a strong relationship with existing Roslyn Analyzers that allow you to inspect both syntax and the semantic models of the compilation. 

# Application Parts in ASP.NET Core MVC

When you create an ASP.NET Core MVC application and add new functionality with either implementations of Controllers and Views or Razor Pages it all just _works_ when you start it. One key element in making all of this possible is a component called the [ApplicationPartsManager](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.applicationparts.applicationpartmanager){:target="_blank"}. This manager will contain a list of [FeatureProviders](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.applicationparts.iapplicationfeatureprovider){:target="_blank"} that will dynamically populate the needed content for e.g. Razor Pages, Controllers, TagHelpers and Views. Read the [Share controllers, views, Razor Pages and more with Application Parts](https://docs.microsoft.com/en-us/aspnet/core/mvc/advanced/app-parts){:target="_blank"} article in the Microsoft documentation for more details on how it works. 

# What is exposed by ASP.NET Core MVC?

The whole way of _automagically_ resolving the content can be really helpful when developing. One big downside is the fact that there is a risk of exposing more than intended. From the main MVC project (the one that constructs the host builder), any referenced project or added package that also reference the [Microsoft.AspNetCore.Mvc](https://www.nuget.org/packages/microsoft.aspnetcore.mvc){:target="_blank"} package will automatically be scanned for controllers. The class needs to match a naming convention that ends with `...Controller` (including inheritance) or is annotated with the `Microsoft.AspNetCore.Mvc.ControllerAttribute`. Each identified controller will then have _all_ it's public methods exposed and keep in mind that _any_ used third-party package has the ability to do exactly the same. It is not unseen that packages get compromised (there are a lot of ways!) by people who have got other and/or bad intentions. Of course it's possible to determine a list of exposed controllers at runtime but I think it's better to have more control (e.g. whitelist of allowed ones) from the start, and earliest possible will of course be at compile time. 

# Determine exposed controllers at compile time

A bit more than a year ago David Fowler shared a [tweet](https://twitter.com/davidfowl/status/1123777569621409793){:target="_blank"} in which he explained that one of the design goals of ASP.NET Core was to remove the scanning of assemblies at start up to find _things_ that are exposed. And that is exactly what the new compiler feature of C# Source Generators will bring to ASP.NET Core MVC. Currently the [samples](https://github.com/dotnet/roslyn-sdk/tree/master/samples/CSharp/SourceGenerators){:target="_blank"} show some basic scenario's on how it can be used, I'm really looking forward on seeing an implementation of it in the ASP.NET Core MVC packages. An additional benefit of all of this is that from static analysis perspective there will be less need to understand what happens at runtime when resolving controllers dynamically, because it's all just part of the compiled assembly.
