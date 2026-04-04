# ZBack CLI — Code-Level Specification

## 1. Purpose

This document defines the exact CLI structure, command parser behavior, argument validation, exit code rules, and implementation shape for ZBack.

This specification is implementation-normative.

---

## 2. Required Entry Files

The ZBack CLI implementation MUST contain at minimum:

```text
/Profiles/ZBack/Cli/
  Program.cs
  ZBackCli.cs
  Commands/
    BuildCommand.cs
    RunCommand.cs
    ValidateCommand.cs
    NewCommand.cs
    DoctorCommand.cs
  Parsing/
    CliParseResult.cs
    CliParser.cs
    CliCommandName.cs
```

File and type names are mandatory for the first implementation.

---

## 3. Required Namespace

All CLI implementation files MUST use this namespace prefix:

```csharp
namespace AEngine.Profiles.ZBack.Cli;
```

Sub-namespaces are allowed under this root:

```csharp
namespace AEngine.Profiles.ZBack.Cli.Commands;
namespace AEngine.Profiles.ZBack.Cli.Parsing;
```

---

## 4. Required Entry Point

`Program.cs` MUST contain exactly one entry class:

```csharp
namespace AEngine.Profiles.ZBack.Cli;

public static class Program
{
    public static int Main(string[] args)
    {
        var cli = new ZBackCli();
        return cli.Execute(args);
    }
}
```

No additional startup behavior is allowed in `Program.Main`.
All logic must be delegated to `ZBackCli`.

---

## 5. Required Main CLI Type

```csharp
namespace AEngine.Profiles.ZBack.Cli;

public sealed class ZBackCli
{
    public int Execute(string[] args)
    {
        ArgumentNullException.ThrowIfNull(args);
        // implementation required
    }
}
```

Rules:
- `Execute()` MUST never throw for normal user errors.
- `Execute()` MUST return process exit code.
- `Execute()` MAY throw only on unrecoverable internal invariant violation.

---

## 6. Supported Commands

The first implementation MUST support exactly these top-level commands:

- `new`
- `build`
- `run`
- `validate`
- `doctor`
- `help`

No aliases are allowed in the first implementation.

---

## 7. Command Parsing Model

### 7.1 Required Enum

```csharp
namespace AEngine.Profiles.ZBack.Cli.Parsing;

public enum CliCommandName
{
    Help,
    New,
    Build,
    Run,
    Validate,
    Doctor,
    Invalid
}
```

### 7.2 Required Parse Result Type

```csharp
namespace AEngine.Profiles.ZBack.Cli.Parsing;

public sealed class CliParseResult
{
    public required CliCommandName CommandName { get; init; }
    public required IReadOnlyList<string> PositionalArguments { get; init; }
    public required IReadOnlyDictionary<string, string> Options { get; init; }
    public required bool IsValid { get; init; }
    public string? ErrorMessage { get; init; }
}
```

### 7.3 Required Parser Type

```csharp
namespace AEngine.Profiles.ZBack.Cli.Parsing;

public sealed class CliParser
{
    public CliParseResult Parse(string[] args)
    {
        ArgumentNullException.ThrowIfNull(args);
        // implementation required
    }
}
```

---

## 8. Parsing Rules

### 8.1 General Rules

- First token is command name.
- Remaining tokens are either positional arguments or options.
- Options MUST use `--name value` form.
- Combined short flags are not supported.
- Unknown command = invalid parse.
- Unknown option = invalid parse.

### 8.2 Empty Args

If `args.Length == 0`, parser MUST return:
- `CommandName = Help`
- `IsValid = true`

### 8.3 Supported Common Options

The first implementation MUST support these options where relevant:

- `--project <path>`
- `--config <name>`
- `--out <path>`
- `--json`

`--json` MUST be represented in `Options` as key=`json`, value=`true`.

---

## 9. Command Dispatch Rules

`ZBackCli.Execute()` MUST perform the following algorithm exactly:

```text
1. Parse args using CliParser.
2. If parse invalid:
   - print error to stderr
   - print short usage
   - return exit code 2
3. Dispatch to command type.
4. Return command exit code.
```

Normative pseudocode:

```text
function Execute(args):
    parseResult = parser.Parse(args)

    if not parseResult.IsValid:
        stderr.write(parseResult.ErrorMessage)
        stderr.write(GetUsage())
        return 2

    switch parseResult.CommandName:
        case Help: return helpCommand.Execute(parseResult)
        case New: return newCommand.Execute(parseResult)
        case Build: return buildCommand.Execute(parseResult)
        case Run: return runCommand.Execute(parseResult)
        case Validate: return validateCommand.Execute(parseResult)
        case Doctor: return doctorCommand.Execute(parseResult)
        default: return 2
```

---

## 10. Required Command Interface

All commands MUST implement:

```csharp
namespace AEngine.Profiles.ZBack.Cli.Commands;

public interface ICliCommand
{
    int Execute(CliParseResult parseResult);
}
```

---

## 11. Required Command Types

The following concrete types MUST exist:

```csharp
public sealed class BuildCommand : ICliCommand
public sealed class RunCommand : ICliCommand
public sealed class ValidateCommand : ICliCommand
public sealed class NewCommand : ICliCommand
public sealed class DoctorCommand : ICliCommand
```

All command classes MUST be parameterless in the first implementation.
Dependencies must be created inside command execution helpers or passed later via explicit construction in `ZBackCli`.

---

## 12. Command Semantics

### 12.1 Help

- prints supported commands
- returns 0

### 12.2 New

- creates project structure
- fails if destination exists and is non-empty

### 12.3 Build

- requires `--project` or one positional project path
- runs ZBack build pipeline

### 12.4 Run

- requires project path
- ensures build exists or triggers build depending on implementation flag
- launches runtime artifact

### 12.5 Validate

- validates project structure and config
- does not compile

### 12.6 Doctor

- validates environment and required tools
- checks compiler executable availability

---

## 13. Exit Codes

ZBack CLI MUST use these exit codes:

- `0` success
- `1` command execution failure caused by project or build problem
- `2` invalid CLI usage or invalid arguments
- `3` internal unexpected failure
- `4` environment failure (missing compiler, inaccessible directory, etc.)

---

## 14. Console Output Rules

- Human-readable messages go to stdout.
- Error messages go to stderr.
- If `--json` is set, command output MUST be valid JSON and human-readable extra text MUST NOT be emitted.

---

## 15. Prohibited Behaviors

The first implementation MUST NOT:
- use reflection-based command discovery;
- silently ignore unknown options;
- mutate parser results after parsing;
- throw on common user mistakes such as missing project path.

---

## 16. Minimal Required Skeleton

```csharp
namespace AEngine.Profiles.ZBack.Cli;

using AEngine.Profiles.ZBack.Cli.Commands;
using AEngine.Profiles.ZBack.Cli.Parsing;

public sealed class ZBackCli
{
    public int Execute(string[] args)
    {
        ArgumentNullException.ThrowIfNull(args);

        var parser = new CliParser();
        var parseResult = parser.Parse(args);

        if (!parseResult.IsValid)
        {
            Console.Error.WriteLine(parseResult.ErrorMessage);
            Console.Error.WriteLine("Use 'zback help' for usage.");
            return 2;
        }

        ICliCommand command = parseResult.CommandName switch
        {
            CliCommandName.Help => new HelpCommandAdapter(),
            CliCommandName.New => new NewCommand(),
            CliCommandName.Build => new BuildCommand(),
            CliCommandName.Run => new RunCommand(),
            CliCommandName.Validate => new ValidateCommand(),
            CliCommandName.Doctor => new DoctorCommand(),
            _ => new InvalidCommandAdapter()
        };

        return command.Execute(parseResult);
    }

    private sealed class HelpCommandAdapter : ICliCommand
    {
        public int Execute(CliParseResult parseResult)
        {
            Console.WriteLine("zback <new|build|run|validate|doctor|help>");
            return 0;
        }
    }

    private sealed class InvalidCommandAdapter : ICliCommand
    {
        public int Execute(CliParseResult parseResult)
        {
            return 2;
        }
    }
}
```

Implementations may refactor helper methods, but they must preserve this structure and behavior.
