# Module Hooks — Code-Level Specification

## 1. Purpose

This document defines the exact hook interfaces that every module implementation must use.

No module may introduce its own lifecycle method names outside this contract.

---

## 2. Required Files

Every module implementation MUST contain:

- `ModuleManifest.json`
- `ModuleEntry.cs`
- optional `Validators/`
- optional `Hooks/`

---

## 3. Required Entry Type

Every module MUST expose exactly one public entry class named:

```csharp
public sealed class ModuleEntry : IModuleEntry
```

No alternative entry class names are allowed in the first implementation.

---

## 4. Required Interface

The exact interface is:

```csharp
public interface IModuleEntry
{
    string Id { get; }
    string Version { get; }
    void Initialize(ModuleInitializationContext context);
    void Validate(ProjectContext context, DiagnosticCollector diagnostics);
    void BeforeCompile(ProjectContext context, DiagnosticCollector diagnostics);
    void AfterCompile(ProjectContext context, BuildArtifactSet artifacts, DiagnosticCollector diagnostics);
}
```

The implementation MUST implement all methods.

---

## 5. Method Rules

### 5.1 Initialize

Purpose:
- receive module services;
- cache immutable references;
- validate internal invariants.

Initialize MUST NOT:
- write files;
- start background threads;
- invoke compiler;
- modify ProjectContext.

### 5.2 Validate

Purpose:
- validate project-level assumptions;
- emit errors and warnings.

Validate MUST:
- be deterministic;
- be side-effect free except diagnostics.

### 5.3 BeforeCompile

Purpose:
- prepare derived data;
- contribute pre-compilation metadata.

BeforeCompile MAY:
- write temporary files in build temp directory only.

BeforeCompile MUST NOT:
- invoke compiler directly.

### 5.4 AfterCompile

Purpose:
- validate artifacts;
- append module metadata;
- run post-build processing permitted by profile.

AfterCompile MUST NOT:
- replace primary build output silently.

---

## 6. Required Supporting Types

### 6.1 ModuleInitializationContext

```csharp
public sealed class ModuleInitializationContext
{
    public required string ModuleRootPath { get; init; }
    public required string ProjectProfileId { get; init; }
    public required IReadOnlyDictionary<string, object> Services { get; init; }
}
```

### 6.2 ProjectContext

```csharp
public sealed class ProjectContext
{
    public required string ProjectRootPath { get; init; }
    public required string ProjectFilePath { get; init; }
    public required string ProfileId { get; init; }
    public required string Configuration { get; init; }
    public required string OutputPath { get; init; }
    public required IReadOnlyList<string> SourceRoots { get; init; }
    public required IReadOnlyDictionary<string, object> Properties { get; init; }
}
```

### 6.3 DiagnosticCollector

```csharp
public sealed class DiagnosticCollector
{
    public void Error(string code, string message, string? file = null);
    public void Warning(string code, string message, string? file = null);
    public void Info(string code, string message, string? file = null);
}
```

### 6.4 BuildArtifactSet

```csharp
public sealed class BuildArtifactSet
{
    public required string OutputRootPath { get; init; }
    public required IReadOnlyList<BuildArtifact> Artifacts { get; init; }
}

public sealed class BuildArtifact
{
    public required string Kind { get; init; }
    public required string Path { get; init; }
}
```

---

## 7. Invocation Order

For every successful build pipeline, hooks MUST be called in this exact order:

1. `Initialize()` once per module after module construction
2. `Validate()` once per module in topological order
3. `BeforeCompile()` once per module in topological order
4. Compiler invocation
5. `AfterCompile()` once per module in reverse topological order

If validation fails, `BeforeCompile()` and `AfterCompile()` MUST NOT be called.

If compiler fails, `AfterCompile()` MUST NOT be called.

---

## 8. Construction Rules

Module instances MUST be created using parameterless constructors only:

```csharp
var module = new ModuleEntry();
```

Dependency injection into constructors is forbidden in first implementation.
All runtime dependencies must be provided via `Initialize()`.

---

## 9. File and Namespace Rules

Required namespace pattern:

```csharp
namespace AEngine.Modules.<ModuleName>;
```

Example:

```csharp
namespace AEngine.Modules.Session;
```

Required file layout:

```text
/<module-root>
  ModuleManifest.json
  ModuleEntry.cs
  /Validators
  /Hooks
```

---

## 10. Error Rules

A module MUST communicate pipeline problems by writing diagnostics, not by printing to console directly.

A module MAY throw only for:
- invalid internal state;
- impossible runtime invariant violation.

A module MUST NOT throw for ordinary project validation errors.
Those must become diagnostics.

---

## 11. Example Skeleton (Normative Shape)

```csharp
namespace AEngine.Modules.Session;

public sealed class ModuleEntry : IModuleEntry
{
    public string Id => "session";
    public string Version => "1.0.0";

    public void Initialize(ModuleInitializationContext context)
    {
        ArgumentNullException.ThrowIfNull(context);
    }

    public void Validate(ProjectContext context, DiagnosticCollector diagnostics)
    {
        ArgumentNullException.ThrowIfNull(context);
        ArgumentNullException.ThrowIfNull(diagnostics);
    }

    public void BeforeCompile(ProjectContext context, DiagnosticCollector diagnostics)
    {
        ArgumentNullException.ThrowIfNull(context);
        ArgumentNullException.ThrowIfNull(diagnostics);
    }

    public void AfterCompile(ProjectContext context, BuildArtifactSet artifacts, DiagnosticCollector diagnostics)
    {
        ArgumentNullException.ThrowIfNull(context);
        ArgumentNullException.ThrowIfNull(artifacts);
        ArgumentNullException.ThrowIfNull(diagnostics);
    }
}
```

Any module implementation that materially deviates from this shape is non-compliant.
