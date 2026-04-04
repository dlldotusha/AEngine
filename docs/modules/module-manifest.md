# Module Manifest — Code-Level Specification

## 1. Purpose

Defines the exact manifest format for modules.

This specification is normative and must be implemented exactly in the first version.

---

## 2. Required File Name

Every module root MUST contain a file named:

```text
ModuleManifest.json
```

No alternative file names are allowed in the first implementation.

---

## 3. Required JSON Schema

Every module manifest MUST serialize to the following structure:

```json
{
  "id": "session",
  "version": "1.0.0",
  "displayName": "Session Module",
  "entryType": "AEngine.Modules.Session.ModuleEntry",
  "supportedProfiles": ["ZBack"],
  "dependencies": [],
  "conflicts": []
}
```

---

## 4. Required Fields

### 4.1 id
- type: string
- required
- unique among modules loaded into one project
- lowercase kebab-case or lowercase single-token name

### 4.2 version
- type: string
- required
- semantic version string

### 4.3 displayName
- type: string
- required
- human-readable module name

### 4.4 entryType
- type: string
- required
- fully qualified type name for `ModuleEntry`

### 4.5 supportedProfiles
- type: array of string
- required
- must contain at least one item

### 4.6 dependencies
- type: array of string
- required
- may be empty

### 4.7 conflicts
- type: array of string
- required
- may be empty

---

## 5. Parsing Contract

The first implementation MUST deserialize to this exact type:

```csharp
namespace AEngine.Modules;

public sealed class ModuleManifest
{
    public required string Id { get; init; }
    public required string Version { get; init; }
    public required string DisplayName { get; init; }
    public required string EntryType { get; init; }
    public required IReadOnlyList<string> SupportedProfiles { get; init; }
    public required IReadOnlyList<string> Dependencies { get; init; }
    public required IReadOnlyList<string> Conflicts { get; init; }
}
```

---

## 6. Validation Rules

The manifest validator MUST validate in this order:

1. file exists
2. JSON is valid
3. all required fields present
4. id not empty
5. version not empty
6. entryType not empty
7. supportedProfiles non-empty
8. dependencies do not contain self id
9. conflicts do not contain self id
10. no duplicate entries in dependencies or conflicts

---

## 7. Failure Behavior

If validation fails:
- module MUST NOT be loaded
- diagnostic MUST be emitted
- build MUST fail if module was explicitly requested by project or profile

---

## 8. Required Minimal Loader Shape

```csharp
namespace AEngine.Modules;

public static class ModuleManifestLoader
{
    public static ModuleManifest Load(string moduleRootPath)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(moduleRootPath);

        var manifestPath = Path.Combine(moduleRootPath, "ModuleManifest.json");
        var json = File.ReadAllText(manifestPath);
        var manifest = JsonSerializer.Deserialize<ModuleManifest>(json)
            ?? throw new InvalidOperationException("Failed to deserialize module manifest.");

        Validate(manifest);
        return manifest;
    }

    private static void Validate(ModuleManifest manifest)
    {
        ArgumentNullException.ThrowIfNull(manifest);

        if (string.IsNullOrWhiteSpace(manifest.Id))
            throw new InvalidOperationException("ModuleManifest.Id is required.");

        if (string.IsNullOrWhiteSpace(manifest.Version))
            throw new InvalidOperationException("ModuleManifest.Version is required.");

        if (string.IsNullOrWhiteSpace(manifest.EntryType))
            throw new InvalidOperationException("ModuleManifest.EntryType is required.");

        if (manifest.SupportedProfiles is null || manifest.SupportedProfiles.Count == 0)
            throw new InvalidOperationException("ModuleManifest.SupportedProfiles must not be empty.");
    }
}
```

The validation method may be expanded, but the order and required checks must be preserved.
