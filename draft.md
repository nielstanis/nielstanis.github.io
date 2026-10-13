---
layout: post
title: MXC - Sandboxing untrusted code from .NET
categories: []
tags: []
draft: true
---

Every .NET developer knows this one, even if we don't like to say it out loud. A NuGet package you pull in runs with the same permissions as your process. It gets your files, your network and your environment variables. We've lived with that for twenty years and call it "dependency management".

Then AI agents showed up and we went one step further: we now let model output run commands on our machines. A prompt injection in a README or a web page, and the agent happily runs `curl | sh` with your permissions.

I've tried to solve this a couple of ways, with .NET libraries inside WebAssembly and with custom `AssemblyLoadContext` implementations <!-- TODO(niels): link your earlier posts on the WebAssembly and AssemblyLoadContext approaches. -->. The `AssemblyLoadContext` route gave me isolation of *loading*, not of *permissions*: it's not a security boundary. WebAssembly does give you a real boundary, but neither is pleasant to adopt. You get different toolchains, extra build steps and a runtime boundary you have to design around. Sandboxing that's hard to adopt doesn't get adopted.

That's why **MXC** (Microsoft eXecution Container) caught my eye. It's open source (MIT), went GA on Windows 11 on October 7, 2026, and powers the local sandboxing in GitHub Copilot CLI. It takes the opposite approach: stay close to your code, with one SDK and a few lines of policy.

A note on how I tested: <!-- TODO(niels): fill in OS, backend and Microsoft.Mxc.Sdk version, e.g. "Ubuntu 24.04 (WSL), bubblewrap, Microsoft.Mxc.Sdk x.y.z" -->. The demos and outputs below are things I ran myself. Everything else, such as backend details and defaults, comes from the project's documentation, and I link to it where it matters.

One scoping point up front, because it matters for the NuGet problem above: MXC contains *processes*, not code inside your process. Your NuGet dependencies are only covered if you move the work that uses them into a contained worker. The rest of this post is about how.

## What it is

MXC is a library you build into your own application to run untrusted code (model output, plugins, tools) in a sandbox on Windows, Linux and macOS. It isn't an agent platform, and installing it doesn't make your machine safe. Your application decides what to run and under which rules. MXC enforces them.

The core is written in Rust, with Rust, .NET (`Microsoft.Mxc.Sdk` on NuGet) and Node SDKs on top. There's also a raw executor (`wxc-exec.exe`) that takes JSON requests, for testing or for cases where you can't embed the SDK. GitHub Copilot CLI uses it for local sandboxing. <!-- TODO(niels): the other adopters (OpenAI Codex, OpenClaw, Replit, LM Studio, NVIDIA OpenShell, Unsloth AI) need a source link; add one or keep this cut. -->That makes sense when nobody wants to build a sandbox per OS.

The flow is simple. Your app builds a versioned request (command, container type, containment rules), MXC validates it and picks a backend, and the workload runs isolated from the rest of the machine. It's the same idea as the Chromium sandbox: don't trust the component that handles untrusted input, and limit what it can touch.

Remember the scoping point: from .NET, your app orchestrates contained child workloads. You can't fence off a single method.

## Backends: it uses what the OS gives you

| Platform | Default backend | Alternatives |
|---|---|---|
| Windows 11 (x64 / ARM64) | `processcontainer` | `windows_sandbox`\*, `wslc`, `microvm`\*, `hyperlight`\*, `isolation_session` |
| Linux (x64 / ARM64) | `bubblewrap` | `lxc`, `microvm`, `hyperlight` |
| macOS (ARM64 / x64) | `seatbelt` | none |

\*Experimental backend.

<!-- TODO(niels): state what GA means on Linux and macOS (the October 7 GA announcement is for Windows 11). -->

Isolation runs from an OS-level process sandbox up to a full VM, all behind one model. But a process container is not a Hyper-V boundary, and the same policy on bubblewrap and on Windows (process container plus WFP for network, default-deny outbound) won't give you identical guarantees. Check what you actually get on *your* target platform.

## The policy is where security happens

Policy covers the filesystem (read-only, read-write, denied paths), the network (proxy and outbound controls) and the UI (clipboard, display, GUI). In C# it's typed objects. This small demo starts from deny:

```csharp
using Microsoft.Mxc.Sdk;
using Microsoft.Mxc.Sdk.V1;

var work = Directory.CreateTempSubdirectory("mxc-demo").FullName;
try
{
    File.WriteAllText(Path.Combine(work, "input.txt"), "hello from the host\n");

    var request = new ContainerRequest(
        "sh -c \"cat input.txt; echo x > blocked.txt && echo WRITE-OK || echo write-denied; " +
        "curl -s -m 3 https://example.com >/dev/null && echo NET-OK || echo net-denied\"")
    {
        Containment = new Containment.Process(),
        Filesystem = new FilesystemPolicy { ReadonlyPaths = { work } },
        Network = new NetworkPolicy
        {
            Egress = new NetworkEgressPolicy { Default = NetworkAction.Deny },
        },
        WorkingDirectory = work,
        TimeoutMs = 30_000,
    };

    try
    {
        var result = await MxcContainer.RunAsync(request);
        Console.Write(result.Stdout);
        Console.Error.Write(result.Stderr);
        foreach (var warning in result.Warnings)
        {
            Console.Error.WriteLine($"warning: {warning}");
        }
        Console.WriteLine($"exit={result.ExitCode} timedOut={result.TimedOut}");
    }
    catch (MxcException error)
    {
        Console.Error.WriteLine($"MXC error [{error.Code}]: {error.Message}");
        return 1;
    }
    return 0;
}
finally
{
    Directory.Delete(work, recursive: true);
}
```

`Containment.Process` means "the host's native process-isolation backend": ProcessContainer on Windows, Bubblewrap on Linux and Seatbelt on macOS ([types reference](https://github.com/microsoft/mxc/blob/main/docs/api-reference/dotnet/v1/types.md)). `Containment.ProcessContainer`, used further down, is the explicit Windows configuration, which is where the extra options such as `CaptureDenials` live. I ran this demo on Linux (bubblewrap) <!-- TODO(niels): explain why, and add the Windows (ProcessContainer) run and output, since Windows is the GA platform -->.

The workload can read the one directory we hand it, can't write to it, and can't reach the network:

```text
hello from the host
write-denied
net-denied
sh: 1: cannot create blocked.txt: Read-only file system
exit=0 timedOut=False
```

`exit=0` is the *shell's* exit code. The sandbox did its job, and the workload just didn't get what it asked for. (The command is Linux-flavored. On Windows, use a `cmd.exe` equivalent.)

### Allowing one host

Default-deny is the easy demo. Real agents need nuget.org or one API endpoint, and that's where policies get sloppy. <!-- TODO(niels): add and test an egress allowlist example (allow a single host, deny everything else) from the NetworkEgressPolicy docs. -->

### Environment variables

The first question I had: does the workload see my secrets (`GITHUB_TOKEN`, cloud credentials)? The request has an `Environment` dictionary and an `InheritDefaultEnvironment` flag ([types reference](https://github.com/microsoft/mxc/blob/main/docs/api-reference/dotnet/v1/types.md)). <!-- TODO(niels): run `env` inside the container with a secret set on the host, with and without InheritDefaultEnvironment, and report exactly what leaks. -->

A sandbox is only as strong as its policy. Give it read-write on your whole home directory and you've built a very elegant container with nothing inside it.

## The .NET SDK

Three main static classes in `Microsoft.Mxc.Sdk.V1` do the work, plus `MxcTelemetry` (covered below) and the `Policy.Filesystem` helpers.

- `MxcContainer` has `Run`/`RunAsync`, which capture stdout and stderr. `Spawn`/`SpawnAsync` give you live pipes, and `SpawnWithPty` gives you an interactive terminal. Be aware that create-and-run PTY support covers IsolationSession, Bubblewrap, LXC and Seatbelt, but not ProcessContainer, so an interactive agent shell on Windows needs a different backend. On Windows, `Probe` evaluates a ProcessContainer request (isolation tier, compatibility diagnostics) without creating a container.
- `MxcLifecycle` has `ProvisionContainer`, `StartContainer`, `StopContainer` and `DeprovisionContainer`, plus `RunInContainer`/`SpawnInContainer` and `Validate*` dry runs.
- `MxcPlatform` has `GetPlatformSupport()` and `GetAvailableBackends()` for feature detection.

That gives you two execution models. You can do a one-shot create-and-run, or a stateful cycle of provision, start, exec, stop and deprovision, which suits reuse such as an agent session.

The policy helpers under `Policy.Filesystem` (`GetAvailableToolsPolicy`, `GetUserProfilePolicy`, `GetTemporaryFilesPolicy`) discover paths from PATH and environment variables. They're handy, and they're also where overbroad grants creep in. `GetUserProfilePolicy` grants *read-only* access to per-user application directories, which sounds harmless until you remember what lives in a profile: SSH keys, cloud CLI config, browser data. Read access is enough to steal those. `GetTemporaryFilesPolicy` deserves even more attention: it grants *read-write* access to the shared host temp directory, which is a cross-process channel and a classic place to plant files.

One gotcha: on `RunAsync`, cancelling the `CancellationToken` only stops *awaiting* the result. The native execution keeps going. Bound the run with `TimeoutMs`, because a `CancellationToken` is not a kill switch.

## Using it without containing your own app

Modern .NET has no in-process isolation (no CAS, no AppDomains), so you push the risky part out of process and contain that. Here are two ways I'd do it:

- Self-relaunch worker. Start your own app via `Environment.ProcessPath` with a worker flag inside MXC and talk to it over stdio. Parsing untrusted files, deserializing third-party data and loading plugins then all happen contained. It's the closest you get to "sandbox this method". It comes with gotchas:
  - Under `dotnet app.dll`, `Environment.ProcessPath` is the `dotnet` host, not your app, so you also need to pass the dll path.
  - The policy needs read access to your app directory, which exposes your config files (`appsettings.json` with connection strings, for example).
  - Treat everything coming back over stdio as untrusted input. The contained worker is now the attacker, so careful deserialization on the host side is the whole point.
- Agent tool execution. With Microsoft.Extensions.AI or Semantic Kernel, send every shell or code-exec tool call through `MxcContainer.RunAsync` with a per-tool policy instead of `Process.Start`.

## Trying to break it

I'm skeptical by profession, so I tried. <!-- TODO(niels): run and report: symlink out of the read-only dir, DNS lookups with egress denied, inherited handles, writing to the shared temp dir. List what held, and what you didn't test. -->

## "My app doesn't work in the sandbox"

At first, it won't, and the documentation says so. We've all seen the sandbox that was "temporarily" disabled in week two and stayed that way until the pentest.

On Windows (ProcessContainer) you can ask MXC to record every denied access:

```csharp
var request = new ContainerRequest(command)
{
    Containment = new Containment.ProcessContainer
    {
        CaptureDenials = new CaptureDenialsPolicy(),
    },
    Filesystem = new FilesystemPolicy { ReadonlyPaths = { inputDir } },
    TimeoutMs = 30_000,
};

var result = await MxcContainer.RunAsync(request);

foreach (var warning in result.Warnings)
{
    Console.Error.WriteLine($"warning: {warning}");
}

if (result.OutputMetadata?.CaptureDenialsError is { } error)
{
    throw new InvalidOperationException(error.Message);
}

var capture = result.OutputMetadata?.CaptureDenials;
Console.WriteLine($"captured {capture?.TotalDenials} denial(s) in {capture?.OutputPath}");
```

By default (`CaptureDenialsMode.Block`, [per the types reference](https://github.com/microsoft/mxc/blob/main/docs/api-reference/dotnet/v1/types.md)) denials are recorded *and* enforced. You get a report of what to add to your policy, and the sandbox stays intact. That's the mode to use in an application. Those recorded denials are also what the Windows-only agent activity report is built on.

`CaptureDenialsMode.Allow` records accesses and lets them through, so nothing is enforced. Use it only to learn what your own trusted tool needs, and never with untrusted code. Then write the policy, switch back to `Block` and test again. Check `BackendCapability.CaptureDenials` to see whether the current host supports it (Windows only).

## What I would check before using it

- Threat model first. What is the untrusted input, and what's the worst the workload can do with the rights you give it?
- Stay on the default backend unless you know why you need another. The experimental ones are marked experimental for a reason.
- Test the denials. Try to read a denied path and reach a blocked host, and make sure both fail.
- Watch the defaults you open. Every allowed path and host is attack surface.
- Expect uneven visibility. Only Windows process containers produce an agent activity report. Session containers, WSLc and microVMs don't.
- Telemetry is off unless the run opts in, the user consents and admin policy allows it. Local open-source builds send nothing to Microsoft, but that doesn't tell you what the shipped NuGet package does. Check it yourself with `MxcTelemetry.GetConsent()` and `MxcTelemetry.GetPolicy()`.
- It's one layer. A sandbox reduces blast radius, but it doesn't make bad code good. You still need least privilege, review and monitoring.

## Conclusion

I'm skeptical by profession, and "new security sandbox" usually sends me looking for the part where it breaks. Still, the approach is sound: deny by default, policy as data, native OS isolation per platform, and honest documentation about the limits. If you're building agents or plugin systems, it's the first step I'd take, ahead of the second, third and fourth steps that the marketing slides skip: least privilege for the host process, review of tool output, and monitoring of denials (see the checklist above).

Documentation and source: <https://github.com/microsoft/mxc>

Further reading:

<!-- TODO(niels): add the official Microsoft announcement and the repo's security/threat-model doc if one exists. -->

- [.NET V1 API signatures](https://github.com/microsoft/mxc/blob/main/docs/api-reference/dotnet/v1/api.md)
- [MXC GA on Windows 11 (Windows Report)](https://windowsreport.com/microsoft-execution-containers-mxc-is-now-generally-available-on-windows-11/)
- [MXC ships before Intune can manage it (beri.net)](https://www.beri.net/article/microsoft-execution-containers-mxc-ga-windows-11-agent-sandbox-intune-entra-agent-365-coming-soon)
- [MXC overview (dsebastien.net)](https://www.dsebastien.net/microsoft-execution-container-mxc/)
- [Running AI agents in sandboxes with MXC (InfoWorld)](https://www.infoworld.com/article/4215416/running-ai-agents-in-sandboxes-with-microsoft-execution-containers.html)
