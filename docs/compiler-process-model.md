# Compiler Process Model — Code-Level Specification

## 1. Purpose

This document defines the exact process-execution model used by ZBack to invoke a compiler executable.

This is a code-level specification.

---

## 2. Required Files

The first implementation MUST contain:

```text
/Profiles/ZBack/
  Compiler/
    CompilerInvocation.cs
    CompilerProcessRunner.cs
    CompilerResult.cs
    CompilerExitCode.cs
```

---

## 3. Required Namespace

```csharp
namespace AEngine.Profiles.ZBack.Compiler;
```

---

## 4. Required Types

### 4.1 CompilerExitCode

```csharp
namespace AEngine.Profiles.ZBack.Compiler;

public enum CompilerExitCode
{
    Success = 0,
    CompilationFailed = 1,
    InvalidInvocation = 2,
    InternalFailure = 3,
    EnvironmentFailure = 4,
    ProcessLaunchFailure = 100,
    Timeout = 101
}
```

### 4.2 CompilerInvocation

```csharp
namespace AEngine.Profiles.ZBack.Compiler;

public sealed class CompilerInvocation
{
    public required string CompilerExecutablePath { get; init; }
    public required string ProjectFilePath { get; init; }
    public required string OutputDirectoryPath { get; init; }
    public required string DiagnosticsFilePath { get; init; }
    public required string ConfigurationName { get; init; }
    public required TimeSpan Timeout { get; init; }
    public bool JsonOutput { get; init; }
}
```

### 4.3 CompilerResult

```csharp
namespace AEngine.Profiles.ZBack.Compiler;

public sealed class CompilerResult
{
    public required CompilerExitCode ExitCode { get; init; }
    public required bool TimedOut { get; init; }
    public required string StandardOutput { get; init; }
    public required string StandardError { get; init; }
    public required string DiagnosticsFilePath { get; init; }
}
```

### 4.4 CompilerProcessRunner

```csharp
namespace AEngine.Profiles.ZBack.Compiler;

public sealed class CompilerProcessRunner
{
    public CompilerResult Run(CompilerInvocation invocation)
    {
        ArgumentNullException.ThrowIfNull(invocation);
        // implementation required
    }
}
```

---

## 5. Required Run Algorithm

Normative pseudocode:

```text
function Run(invocation):
    validate invocation paths

    start process with:
        fileName = invocation.CompilerExecutablePath
        arguments = build compiler args
        redirect stdout = true
        redirect stderr = true
        use shell execute = false
        create no window = true

    wait up to invocation.Timeout

    if timeout:
        kill process tree
        return Timeout result

    capture stdout
    capture stderr
    map exit code
    return CompilerResult
```

---

## 6. Required ProcessStartInfo Settings

Implementation MUST set:

```csharp
new ProcessStartInfo
{
    FileName = invocation.CompilerExecutablePath,
    Arguments = BuildArguments(invocation),
    RedirectStandardOutput = true,
    RedirectStandardError = true,
    UseShellExecute = false,
    CreateNoWindow = true,
    WorkingDirectory = Path.GetDirectoryName(invocation.ProjectFilePath)!
}
```

No deviation is allowed in first implementation.

---

## 7. Required Argument Builder

The exact argument order MUST be:

```text
compile --project <project> --out <out> --diagnostics <diagnostics> --config <config>
```

If `JsonOutput == true`, append:

```text
--json
```

Required helper:

```csharp
private static string BuildArguments(CompilerInvocation invocation)
```

---

## 8. Timeout Rules

- Timeout MUST be enforced using wall-clock time.
- On timeout, runner MUST attempt to kill the process.
- On timeout, runner MUST return `CompilerExitCode.Timeout`.
- Timeout MUST be treated as hard failure by build pipeline.

---

## 9. Exit Code Mapping

If process exits normally, map integer code as follows:

```csharp
private static CompilerExitCode MapExitCode(int rawExitCode)
{
    return rawExitCode switch
    {
        0 => CompilerExitCode.Success,
        1 => CompilerExitCode.CompilationFailed,
        2 => CompilerExitCode.InvalidInvocation,
        3 => CompilerExitCode.InternalFailure,
        4 => CompilerExitCode.EnvironmentFailure,
        _ => CompilerExitCode.InternalFailure
    };
}
```

---

## 10. Launch Failure Rules

If process cannot start due to:
- missing executable
- invalid path
- OS process start failure

then runner MUST return:
- `CompilerExitCode.ProcessLaunchFailure`
- `TimedOut = false`
- captured stderr MAY be empty

Runner MUST NOT throw for launch failure.

---

## 11. Output Reading Rules

The first implementation MUST use:

```csharp
var stdout = process.StandardOutput.ReadToEnd();
var stderr = process.StandardError.ReadToEnd();
```

Output MUST be read fully before returning.

Future async streaming is out of scope.

---

## 12. Validation Rules

Before starting process, runner MUST validate:
- compiler executable path not null/empty
- project file path not null/empty
- output directory path not null/empty
- diagnostics file path not null/empty
- timeout > TimeSpan.Zero

Invalid invocation object MUST throw `ArgumentException`.

This is an internal programming error, not a user error.

---

## 13. Minimal Required Skeleton

```csharp
namespace AEngine.Profiles.ZBack.Compiler;

using System.Diagnostics;

public sealed class CompilerProcessRunner
{
    public CompilerResult Run(CompilerInvocation invocation)
    {
        ArgumentNullException.ThrowIfNull(invocation);

        if (string.IsNullOrWhiteSpace(invocation.CompilerExecutablePath))
            throw new ArgumentException("CompilerExecutablePath is required.", nameof(invocation));

        if (invocation.Timeout <= TimeSpan.Zero)
            throw new ArgumentException("Timeout must be positive.", nameof(invocation));

        try
        {
            using var process = new Process();
            process.StartInfo = new ProcessStartInfo
            {
                FileName = invocation.CompilerExecutablePath,
                Arguments = BuildArguments(invocation),
                RedirectStandardOutput = true,
                RedirectStandardError = true,
                UseShellExecute = false,
                CreateNoWindow = true,
                WorkingDirectory = Path.GetDirectoryName(invocation.ProjectFilePath)!
            };

            process.Start();

            if (!process.WaitForExit((int)invocation.Timeout.TotalMilliseconds))
            {
                try { process.Kill(entireProcessTree: true); } catch { }

                return new CompilerResult
                {
                    ExitCode = CompilerExitCode.Timeout,
                    TimedOut = true,
                    StandardOutput = string.Empty,
                    StandardError = string.Empty,
                    DiagnosticsFilePath = invocation.DiagnosticsFilePath
                };
            }

            var stdout = process.StandardOutput.ReadToEnd();
            var stderr = process.StandardError.ReadToEnd();

            return new CompilerResult
            {
                ExitCode = MapExitCode(process.ExitCode),
                TimedOut = false,
                StandardOutput = stdout,
                StandardError = stderr,
                DiagnosticsFilePath = invocation.DiagnosticsFilePath
            };
        }
        catch
        {
            return new CompilerResult
            {
                ExitCode = CompilerExitCode.ProcessLaunchFailure,
                TimedOut = false,
                StandardOutput = string.Empty,
                StandardError = string.Empty,
                DiagnosticsFilePath = invocation.DiagnosticsFilePath
            };
        }
    }

    private static string BuildArguments(CompilerInvocation invocation)
    {
        var args = $"compile --project \"{invocation.ProjectFilePath}\" --out \"{invocation.OutputDirectoryPath}\" --diagnostics \"{invocation.DiagnosticsFilePath}\" --config \"{invocation.ConfigurationName}\"";
        if (invocation.JsonOutput)
            args += " --json";
        return args;
    }

    private static CompilerExitCode MapExitCode(int rawExitCode)
    {
        return rawExitCode switch
        {
            0 => CompilerExitCode.Success,
            1 => CompilerExitCode.CompilationFailed,
            2 => CompilerExitCode.InvalidInvocation,
            3 => CompilerExitCode.InternalFailure,
            4 => CompilerExitCode.EnvironmentFailure,
            _ => CompilerExitCode.InternalFailure
        };
    }
}
```

Any materially different process model is non-compliant in first implementation.
