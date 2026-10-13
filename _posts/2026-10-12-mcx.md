---
layout: post
title: MXC - Microsoft's sandbox for code you shouldn't trust 
categories: []
tags: []
---

# But are going to run anyway...

Every .NET developer knows this one, even if we don't like to say it out loud. A NuGet package you pull in runs with the same permissions as your process. It gets your files, your network and your environment variables. We've lived with that for twenty years and call it "dependency management".

Then AI agents showed up and we went one step further: we now let model output run commands on our machines. What could go wrong? A lot.

I've tried to solve this a couple of ways, with .NET libraries inside WebAssembly and with custom `AssemblyLoadContext` implementations. Both work, but neither is pleasant to adopt. You get different toolchains, extra build steps and a runtime boundary you have to design around. Sandboxing that's hard to adopt doesn't get adopted.

That's why **MXC** (Microsoft eXecution Container) caught my eye. It's open source (MIT), went GA on Windows 11 on October 7, 2026, and powers the local sandboxing in GitHub Copilot CLI. It takes the opposite approach: stay close to your code, with one SDK and a few lines of policy. It's very new, so most of what follows comes from the project's own documentation.

## What it is

MXC is a library you build into your own application to run untrusted code (model output, plugins, tools) in a sandbox on Windows, Linux and macOS. It isn't an agent platform, and installing it doesn't make your machine safe. Your application decides what to run and under which rules. MXC enforces them.

The core is written in Rust, with Rust, .NET (`Microsoft.Mxc.Sdk` on NuGet) and Node SDKs on top. There's also a raw executor (`wxc-exec.exe`) that takes JSON requests, for testing or for cases where you can't embed the SDK. OpenAI Codex, GitHub Copilot, OpenClaw, Replit, LM Studio, NVIDIA OpenShell and Unsloth AI all support it, which makes sense when none of them wants to build a sandbox per OS.

The flow is simple. Your app builds a versioned request (command, container type, containment rules), MXC validates it and picks a backend, and the workload runs isolated from the rest of the machine. It's the same idea as the Chromium sandbox: don't trust the component that handles untrusted input, and limit what it can touch.

One thing to keep in mind: MXC contains *processes*, not code inside your process. From .NET, your app orchestrates contained child workloads. You can't fence off a single method.

## Backends: it uses what the OS gives you

| Platform | Default backend | Alternatives |
|---|---|---|
| Windows 11 (x64 / ARM64) | `processcontainer` | `windows_sandbox`\*, `wslc`, `microvm`\*, `hyperlight`\*, `isolation_session` |
| Linux (x64 / ARM64) | `bubblewrap` | `lxc`, `microvm`, `hyperlight` |
| macOS (ARM64 / x64) | `seatbelt` | none |

\* experimental

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

The workload can read the one directory we hand it, can't write to it, and can't reach the network:

```text
hello from the host
write-denied
net-denied
sh: 1: cannot create blocked.txt: Read-only file system
exit=0 timedOut=False
```

`exit=0` is the *shell's* exit code. The sandbox did its job, and the workload just didn't get what it asked for. (The command is Linux-flavored. On Windows, use a `cmd.exe` equivalent.)



A sandbox is only as strong as its policy. Give it read-write on your whole home directory and you've built a very elegant container with nothing inside it.

## The .NET SDK

Three static classes in `Microsoft.Mxc.Sdk.V1` do the work.

- `MxcContainer` has `Run`/`RunAsync`, which capture stdout and stderr. `Spawn`/`SpawnAsync` give you live pipes, and `SpawnWithPty` gives you an interactive terminal. On Windows, `Probe` evaluates a ProcessContainer request (isolation tier, compatibility diagnostics) without creating a container.
- `MxcLifecycle` has `ProvisionContainer`, `StartContainer`, `StopContainer` and `DeprovisionContainer`, plus `RunInContainer`/`SpawnInContainer` and `Validate*` dry runs.
- `MxcPlatform` has `GetPlatformSupport()` and `GetAvailableBackends()` for feature detection.

That gives you two execution models. You can do a one-shot create-and-run, or a stateful cycle of provision, start, exec, stop and deprovision, which suits reuse such as an agent session.

The policy helpers under `Policy.Filesystem` (`GetAvailableToolsPolicy`, `GetUserProfilePolicy`, `GetTemporaryFilesPolicy`) discover paths from PATH and environment variables. They're handy, and they're also where overbroad grants creep in. Naturally, the convenient helper is the one that hands out your user profile.

One gotcha: on `RunAsync`, cancelling the `CancellationToken` only stops *awaiting* the result. The native execution keeps going. Bound the run with `TimeoutMs`, because a `CancellationToken` is not a kill switch.

## Using it without containing your own app

Modern .NET has no in-process isolation (no CAS, no AppDomains), so you push the risky part out of process and contain that. Here are three ways I'd do it:

- Self-relaunch worker. Start your own app via `Environment.ProcessPath` with a worker flag inside MXC and talk to it over stdio. Parsing untrusted files, deserializing third-party data and loading plugins then all happen contained. It's the closest you get to "sandbox this method".
- Agent tool execution. With Microsoft.Extensions.AI or Semantic Kernel, send every shell or code-exec tool call through `MxcContainer.RunAsync` with a per-tool policy instead of `Process.Start`.

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

By default (`CaptureDenialsMode.Block`) denials are recorded *and* enforced. You get a report of what to add to your policy, and the sandbox stays intact. That's the mode to use in an application.

`CaptureDenialsMode.Allow` records accesses and lets them through, so nothing is enforced. Use it only to learn what your own trusted tool needs, and never with untrusted code. Then write the policy, switch back to `Block` and test again. Check `BackendCapability.CaptureDenials` to see whether the current host supports it (Windows only).

## What I would check before using it

- Threat model first. What is the untrusted input, and what's the worst the workload can do with the rights you give it?
- Stay on the default backend unless you know why you need another. The experimental ones are marked experimental for a reason.
- Test the denials. Try to read a denied path and reach a blocked host, and make sure both fail.
- Watch the defaults you open. Every allowed path and host is attack surface.
- Expect uneven visibility. Only Windows process containers produce an agent activity report. Session containers, WSLc and microVMs don't.
- Telemetry is off unless the run opts in, the user consents and admin policy allows it. Local open-source builds send nothing to Microsoft. Verify that in your own build.
- It's one layer. A sandbox reduces blast radius, but it doesn't make bad code good. You still need least privilege, review and monitoring.

## Conclusion

I'm skeptical by profession, and "new security sandbox" usually sends me looking for the part where it breaks. Still, the approach is sound: deny by default, policy as data, native OS isolation per platform, and honest documentation about the limits. If you're building agents or plugin systems, it's the first step I'd take, ahead of the second, third and fourth steps that the marketing slides skip.

Documentation and source: <https://github.com/microsoft/mxc>

Further reading:

- [.NET V1 API signatures](https://github.com/microsoft/mxc/blob/main/docs/api-reference/dotnet/v1/api.md)
- [MXC GA on Windows 11 (Windows Report)](https://windowsreport.com/microsoft-execution-containers-mxc-is-now-generally-available-on-windows-11/)
- [MXC ships before Intune can manage it (beri.net)](https://www.beri.net/article/microsoft-execution-containers-mxc-ga-windows-11-agent-sandbox-intune-entra-agent-365-coming-soon)
- [MXC overview (dsebastien.net)](https://www.dsebastien.net/microsoft-execution-container-mxc/)
- [Running AI agents in sandboxes with MXC (InfoWorld)](https://www.infoworld.com/article/4215416/running-ai-agents-in-sandboxes-with-microsoft-execution-containers.html)
