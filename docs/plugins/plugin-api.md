# Plugin API — Code-Level Specification

## 1. Purpose

This document defines the exact code-level plugin contract for ACore.

Plugins are sandboxed developer-environment extensions.
They extend tooling and UI only.
They must not alter build output, project semantics, compiler behavior, or ACore internals.

This document is normative.

---

## 2. Required Files

Every plugin package MUST contain at minimum:

```text
/<plugin-root>
  plugin.json
  PluginEntry.cs
```

Optional directories:

```text
  Panels/
  Inspectors/
  Commands/
```

---

## 3. Required Namespace Pattern

Plugins MUST use the namespace prefix:

```csharp
namespace AEngine.Plugins.<PluginName>;
```

Example:

```csharp
namespace AEngine.Plugins.ColorPreview;
```

---

## 4. Required Entry Type

Each plugin MUST expose exactly one public entry type named `PluginEntry`:

```csharp
public sealed class PluginEntry : IPluginEntry
```

Alternative entry type names are forbidden in the first implementation.

---

## 5. Required Interface

```csharp
namespace AEngine.PluginModel;

public interface IPluginEntry
{
    string Id { get; }
    string Version { get; }
    void Initialize(PluginContext context);
}
```

All plugins MUST implement this interface exactly.

---

## 6. Required Context Type

```csharp
namespace AEngine.PluginModel;

public sealed class PluginContext
{
    public required string PluginRootPath { get; init; }
    public required string PluginId { get; init; }
    public required string PluginVersion { get; init; }
    public required IPluginPermissions Permissions { get; init; }
    public required IPluginUiApi Ui { get; init; }
    public required IPluginDiagnosticsApi Diagnostics { get; init; }
    public required IPluginMetadataApi Metadata { get; init; }
}
```

No mutable setters are allowed after initialization.

---

## 7. Required Permission API

```csharp
namespace AEngine.PluginModel;

public interface IPluginPermissions
{
    bool IsAllowed(string capability);
    void Demand(string capability);
}
```

Rules:
- `IsAllowed()` returns capability state.
- `Demand()` MUST throw `PluginPermissionDeniedException` if not allowed.
- All privileged plugin APIs MUST call `Demand()` internally before performing work.

Required exception:

```csharp
namespace AEngine.PluginModel;

public sealed class PluginPermissionDeniedException : Exception
{
    public PluginPermissionDeniedException(string pluginId, string capability)
        : base($"Plugin '{pluginId}' is not allowed to use capability '{capability}'.")
    {
    }
}
```

---

## 8. Required UI API

```csharp
namespace AEngine.PluginModel;

public interface IPluginUiApi
{
    void RegisterPanel(PluginPanelDescriptor panel);
    void RegisterCommand(PluginCommandDescriptor command);
}
```

Required descriptors:

```csharp
namespace AEngine.PluginModel;

public sealed class PluginPanelDescriptor
{
    public required string Id { get; init; }
    public required string Title { get; init; }
    public required Action Render { get; init; }
}

public sealed class PluginCommandDescriptor
{
    public required string Id { get; init; }
    public required string DisplayName { get; init; }
    public required Action Execute { get; init; }
}
```

Rules:
- `RegisterPanel()` MUST internally require capability `UI.Panel.Create`.
- `RegisterCommand()` MUST internally require capability `UI.Command.Register`.

---

## 9. Required Diagnostics API

```csharp
namespace AEngine.PluginModel;

public interface IPluginDiagnosticsApi
{
    void Info(string code, string message);
    void Warning(string code, string message);
    void Error(string code, string message);
}
```

Rules:
- Plugins MUST report errors through this API.
- Plugins MUST NOT write directly to the console.

---

## 10. Required Metadata API

```csharp
namespace AEngine.PluginModel;

public interface IPluginMetadataApi
{
    IReadOnlyDictionary<string, string> GetProjectMetadata();
}
```

Rules:
- Access MUST require capability `Metadata.Read`.
- Returned collection MUST be immutable from plugin point of view.

---

## 11. Initialization Rules

Plugin loading sequence MUST be:

1. Discover `plugin.json`
2. Validate schema
3. Resolve permissions by plugin id + version
4. Construct `PluginEntry` using parameterless constructor
5. Construct `PluginContext`
6. Call `Initialize(context)`

`Initialize()` MUST be called exactly once.

---

## 12. Construction Rules

Plugins MUST be created using parameterless constructor only:

```csharp
var plugin = new PluginEntry();
```

Constructor injection is forbidden in first implementation.
All dependencies are provided via `PluginContext`.

---

## 13. Allowed Behaviors

Plugins MAY:
- register UI panels;
- register commands;
- emit diagnostics;
- read allowed metadata.

Plugins MUST NOT:
- spawn uncontrolled background processes;
- invoke compiler directly;
- modify project files unless a future API explicitly allows it;
- access filesystem arbitrarily;
- use reflection to inspect ACore internals.

---

## 14. Failure Rules

If a plugin throws during `Initialize()`:
- plugin MUST be marked Faulted
- diagnostic MUST be emitted
- ACore startup MUST continue

If plugin callback throws after initialization:
- callback failure MUST be caught at plugin-host boundary
- diagnostic MUST be emitted
- plugin remains loaded unless policy says otherwise

---

## 15. Required Minimal Skeleton

```csharp
namespace AEngine.Plugins.ColorPreview;

using AEngine.PluginModel;

public sealed class PluginEntry : IPluginEntry
{
    public string Id => "color-preview";
    public string Version => "1.0.0";

    public void Initialize(PluginContext context)
    {
        ArgumentNullException.ThrowIfNull(context);

        context.Ui.RegisterPanel(new PluginPanelDescriptor
        {
            Id = "color-preview.panel",
            Title = "Color Preview",
            Render = () =>
            {
                context.Diagnostics.Info("PLG0001", "Render called.");
            }
        });
    }
}
```

Any implementation materially different from this shape is non-compliant.
