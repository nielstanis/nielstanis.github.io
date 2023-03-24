---
layout: post
title: Sandboxing and isolating .NET assemblies
categories: [sandbox]
tags: [csharp, dotnet, wasm, wasi]
---

A week ago Microsoft released a preview of a new compiler feature called [C# Source Generators](https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/){:target="_blank"}. This feature allows to inspect existing source-code and generate additional C# source files that will be added to the compiled output. One of the goals is to use this in order to remove the need for any (slower) dynamic runtime behavior that rely on reflection. There are also benefits from security perspective that can be added to that list, let's look into that a bit further! 

# Source Generator

When a Source Generator is added to a .NET Core project it will be executed when the project is being build. It has the ability to inspect the programs source artifacts and create additional files that will be added to the output. The concept is also called _Compile Time Metaprogramming_ as explained in the [design document](https://github.com/dotnet/roslyn/blob/master/docs/features/source-generators.md){:target="_blank"}. Source Generators have a strong relationship with existing Roslyn Analyzers that allow you to inspect both syntax and the semantic models of the compilation. 