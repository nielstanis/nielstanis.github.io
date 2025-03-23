---
layout: talk
title: Using WebAssembly to run, extend, and secure your .NET application
---

WebAssembly (WASM) has come a long way since its first release in 2017. As a technology stack running inside the web browser, it even allows products like Adobe Photoshop to run in that context, and with Blazor WebAssembly .NET runs inside of the browser as well. Now, WASM is expanding beyond the browser to run in a server-based context. With the introduction of WebAssembly System Interface (WASI), the technology leverages a standardised API that allows it to run on any system that supports it, for example to support cloud-based workloads.
Had WASM and WASI been around in 2009, Docker would not have existed according to one of its founders, Solomon Hykes. WASM has a strong security posture given how it works with linear memory space and how it supports a sandboxed-based environment called “nano-process”, which uses a capabilities-based security model. Users can even take .NET and, with the help of WASI, run it on a Trusted Execution Environment (TEE) to add an additional layer of security.
In this session we'll start out with going through some of the basic security features of WASM and then move to running and extending an .NET application it with WASM module written in Rust. After that we'll focus on the security features and run .NET on a TEE and use the sandbox and the capabilities based security model to limit what it's allowed to do.

## Videos

[![Future Tech 2023: Niels Tanis - Using WebAssembly to run, extend, and secure your .NET application](https://img.youtube.com/vi/6W25YjFFpJY/0.jpg)](https://www.youtube.com/watch?v=6W25YjFFpJY "Future Tech 2023: Niels Tanis - Using WebAssembly to run, extend, and secure your .NET application")

## Links

- Swetugg 2024 Göteborg - [Using WebAssembly to run, extend, and secure your .NET application](https://swetugg.se/gbg-2024/speakers/niels-tanis#using-webassembly-to-run-extend-and-secure-your-net-application)
- DotNetFriday - 24th of May 2024 - [Using WebAssembly to run, extend, and secure your .NET application](https://dotnetfriday.nl/)
- Swetugg 2024 Stockholm - [Using WebAssembly to run, extend, and secure your .NET application](https://swetugg.- OWASP BeNeLux 2023 - [Using WebAssembly to run, extend, and secure your application](https://www.owaspbenelux.eu/program/talks#Niels-Tanis)
- OWASP 2023 Global AppSec Washington, DC - [Using WebAssembly to run, extend, and secure your application](https://owasp2023globalappsecwashin.sched.com/event/1OM3A/using-webassembly-to-run-extend-and-secure-your-application)
- NDC Porto 2023 - [Using WebAssembly to run, extend, and secure your .NET application](https://ndcporto.com/agenda/using-webassembly-to-run-extend-and-secure-your-net-application-0njh/0c3ybbuclrf)
- WeAreDevelopers World Congress 2023 - Using WebAssembly to run, extend, and secure your application
- Code Europe 2023 - Crawow & Warsaw - Using WebAssembly to run, extend, and secure your .NET application
- FutureTech 2023 - Using WebAssembly to run, extend, and secure your .NET application - [Slides](https://github.com/nielstanis/FutureTech2023/blob/main/Slides/FutureTech2023-WASM.pdf) - [GitHub](https://github.com/nielstanis/FutureTech2023)
- NDC Security 2023 - [Using WebAssembly to run, extend, and secure your .NET application](https://ndc-security.com/agenda/using-webassembly-to-run-extend-and-secure-your-net-application-0dlk/0osljmitryd)
- UpdateConference 2022 - [Using WebAssembly to run, extend, and secure your .NET application](https://www.updateconference.net/en/2022/session/using-webassembly-to-run--extend--and-secure-your--net-application)
