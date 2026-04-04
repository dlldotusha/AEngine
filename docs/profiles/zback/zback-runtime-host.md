# ZBack Runtime Host — Code-Level Specification

## 1. Purpose

This document defines the exact runtime host implementation used by ZBack to execute a built ZBack project.

This specification is normative.

---

## 2. Required Files

The first implementation MUST contain:

```text
/Profiles/ZBack/Runtime/
  RuntimeHost.cs
  RuntimeHostOptions.cs
  ConnectionLoop.cs
  ExecutionContextLoop.cs
```

---

## 3. Required Namespace

```csharp
namespace AEngine.Profiles.ZBack.Runtime;
```

---

## 4. Required Types

### 4.1 RuntimeHostOptions

```csharp
namespace AEngine.Profiles.ZBack.Runtime;

public sealed class RuntimeHostOptions
{
    public required string ArtifactRootPath { get; init; }
    public required TimeSpan TickInterval { get; init; }
    public required IExecutionContextFactory ExecutionContextFactory { get; init; }
    public required IConnectionSource ConnectionSource { get; init; }
}
```

### 4.2 IConnectionSource

```csharp
namespace AEngine.Profiles.ZBack.Runtime;

public interface IConnectionSource
{
    IRuntimeConnection WaitForNextConnection(CancellationToken cancellationToken);
}
```

### 4.3 IRuntimeConnection

```csharp
namespace AEngine.Profiles.ZBack.Runtime;

public interface IRuntimeConnection : IDisposable
{
    string Id { get; }
    bool IsActive { get; }
}
```

### 4.4 RuntimeHost

```csharp
namespace AEngine.Profiles.ZBack.Runtime;

public sealed class RuntimeHost
{
    public void Run(RuntimeHostOptions options, CancellationToken cancellationToken)
    {
        ArgumentNullException.ThrowIfNull(options);
        // implementation required
    }
}
```

---

## 5. Required Runtime Algorithm

Normative pseudocode:

```text
function Run(options, cancellationToken):
    validate options

    while cancellationToken not cancelled:
        connection = options.ConnectionSource.WaitForNextConnection(cancellationToken)
        runConnection(connection, options)
```

This means the first implementation is sequential and single-connection by default.

---

## 6. Required Connection Handling Algorithm

Normative pseudocode:

```text
function runConnection(connection, options):
    ctx = options.ExecutionContextFactory.CreateNew()
    ctx.Initialize()

    while connection.IsActive:
        ctx.Tick()
        sleep(options.TickInterval)

    ctx.Shutdown()
    connection.Dispose()
```

---

## 7. Exact Behavioral Requirements

The runtime host MUST:
- create exactly one new execution context per accepted connection;
- initialize that context exactly once;
- call Tick() repeatedly while connection remains active;
- call Shutdown() exactly once before disposing connection;
- process one connection at a time in the first implementation.

---

## 8. Tick Rules

- Tick interval MUST come from `RuntimeHostOptions.TickInterval`.
- Interval MUST be positive.
- First implementation MUST use `Thread.Sleep()` or equivalent simple wall-clock delay.
- Advanced schedulers are out of scope.

---

## 9. Error Handling Rules

If `CreateNew()` throws:
- runtime host MUST dispose connection;
- runtime host MUST continue outer accept loop.

If `Initialize()` throws:
- runtime host MUST attempt `Shutdown()` only if context creation succeeded;
- runtime host MUST dispose connection;
- runtime host MUST continue outer accept loop.

If `Tick()` throws:
- runtime host MUST call `Shutdown()`;
- runtime host MUST dispose connection;
- runtime host MUST continue outer accept loop.

If `Shutdown()` throws:
- runtime host MUST still dispose connection and continue.

---

## 10. Required Minimal Skeleton

```csharp
namespace AEngine.Profiles.ZBack.Runtime;

public sealed class RuntimeHost
{
    public void Run(RuntimeHostOptions options, CancellationToken cancellationToken)
    {
        ArgumentNullException.ThrowIfNull(options);

        if (options.TickInterval <= TimeSpan.Zero)
            throw new ArgumentException("TickInterval must be positive.", nameof(options));

        while (!cancellationToken.IsCancellationRequested)
        {
            var connection = options.ConnectionSource.WaitForNextConnection(cancellationToken);
            RunSingleConnection(connection, options);
        }
    }

    private static void RunSingleConnection(IRuntimeConnection connection, RuntimeHostOptions options)
    {
        IExecutionContext? context = null;

        try
        {
            context = options.ExecutionContextFactory.CreateNew();
            context.Initialize();

            while (connection.IsActive)
            {
                context.Tick();
                Thread.Sleep(options.TickInterval);
            }
        }
        finally
        {
            try
            {
                context?.Shutdown();
            }
            catch
            {
            }

            connection.Dispose();
        }
    }
}
```

Any materially different first-implementation runtime loop is non-compliant.
