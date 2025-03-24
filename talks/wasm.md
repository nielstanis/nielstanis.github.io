---
layout: talk
title: Using WebAssembly to run, extend, and secure your .NET application
---

WebAssembly (WASM) has come a long way since its first release in 2017. As a technology stack running inside the web browser, it even allows products like Adobe Photoshop to run in that context, and with Blazor WebAssembly .NET runs inside of the browser as well. Now, WASM is expanding beyond the browser to run in a server-based context. With the introduction of WebAssembly System Interface (WASI), the technology leverages a standardised API that allows it to run on any system that supports it, for example to support cloud-based workloads.
Had WASM and WASI been around in 2009, Docker would not have existed according to one of its founders, Solomon Hykes. WASM has a strong security posture given how it works with linear memory space and how it supports a sandboxed-based environment called “nano-process”, which uses a capabilities-based security model. Users can even take .NET and, with the help of WASI, run it on a Trusted Execution Environment (TEE) to add an additional layer of security.
In this session we'll start out with going through some of the basic security features of WASM and then move to running and extending an .NET application it with WASM module written in Rust. After that we'll focus on the security features and run .NET on a TEE and use the sandbox and the capabilities based security model to limit what it's allowed to do.

## Videos

[![Swetugg 2024 Göteborg: Niels Tanis - Using WebAssembly to run, extend, and secure your .NET application](https://img.youtube.com/vi/mINJd5Zc7mE/0.jpg)](https://www.youtube.com/watch?v=mINJd5Zc7mE "Swetugg 2024 Göteborg: Niels Tanis - Using WebAssembly to run, extend, and secure your .NET application")
[![Future Tech 2023: Niels Tanis - Using WebAssembly to run, extend, and secure your .NET application](https://img.youtube.com/vi/6W25YjFFpJY/0.jpg)](https://www.youtube.com/watch?v=6W25YjFFpJY "Future Tech 2023: Niels Tanis - Using WebAssembly to run, extend, and secure your .NET application")

## Events

- Swetugg 2024 Göteborg - [Slides](https://github.com/nielstanis/swetugg2024wasm/blob/main/Slides/SWETUGG2024.pdf) - [GitHub](https://github.com/nielstanis/swetugg2024wasm/)
- DotNetFriday - 24th of May 2024 - [Slides](https://github.com/nielstanis/dotnetfriday2024/blob/main/Slides/DotNetFriday2024.pdf) - [GitHub](https://github.com/nielstanis/dotnetfriday2024/)
- Swetugg 2024 Stockholm - [Slides](https://github.com/nielstanis/swetugg2024/blob/main/Slides/SWETUGG2024-WASM.pdf) - [GitHub](https://github.com/nielstanis/swetugg2024/)
- OWASP BeNeLux 2023 - [Slides](https://github.com/nielstanis/OWASPBeNeLux2023/blob/main/slides/OWASPBenelux.pdf) - [GitHub](https://github.com/nielstanis/OWASPBeNeLux2023/)
- OWASP 2023 Global AppSec Washington, DC - [Slides](https://github.com/nielstanis/AppSecDC2023/blob/main/slides/AppSecDC2023-Wasm.pdf) - [GitHub](https://github.com/nielstanis/AppSecDC2023/)
- NDC Porto 2023 - [Slides](https://github.com/nielstanis/ndcporto2023-wasm/blob/main/slides/NDCPorto2023-WASM.pdf) - [GitHub](https://github.com/nielstanis/ndcporto2023-wasm/)
- WeAreDevelopers World Congress 2023 - [Slides](https://github.com/nielstanis/WeAreDevs2023/blob/main/Slides/WeAreDevs-WASM.pdf) - [GitHub](https://github.com/nielstanis/WeAreDevs2023/)
- Code Europe 2023 - Crawow & Warsaw - [Slides](https://github.com/nielstanis/CodeEurope2023-WASM/blob/main/Slides/CodeEurope2023-WASM.pdf) - [GitHub](https://github.com/nielstanis/CodeEurope2023-WASM/)
- FutureTech 2023 - [Slides](https://github.com/nielstanis/FutureTech2023/blob/main/Slides/FutureTech2023-WASM.pdf) - [GitHub](https://github.com/nielstanis/FutureTech2023/)
- NDC Security 2023 - [Slides](https://github.com/nielstanis/NDCSecurity2023/blob/main/Slides/NDCSecurity2023-WASM.pdf) - [GitHub](https://github.com/nielstanis/NDCSecurity2023/)
- UpdateConference 2022 - [Slides](https://github.com/nielstanis/updateconference2022/blob/main/Slides/UpdateConference-WASM.pdf) - [GitHub](https://github.com/nielstanis/updateconference2022/)
