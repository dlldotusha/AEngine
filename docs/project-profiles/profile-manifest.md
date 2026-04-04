# Project Profile Manifest — Code-Level Specification

## 1. Purpose

This document defines the exact manifest format for all Project Profiles.

Every profile MUST provide one manifest file.
This file is the canonical metadata source used by ACore and standalone tooling.

---

## 2. Required File Name

The manifest file name MUST be:

```text
ProfileManifest.json
```

---

## 3. Required File Location

Each profile MUST live in its own root directory.
The manifest MUST be placed at the profile root.

Required layout:

```text
/<profile-root>
  ProfileManifest.json
```

---

## 4. Required Schema

The manifest MUST deserialize into the following shape:

```json
{
  "id": "ZBack",
  "version": "1.0.0",
  "displayName": "ZBack",
  "toolchainExecutable": "zback",
  "compilerExecutable": "zcompiler",
  "builtInModules": ["connection", "session"],
  "projectFileName": "project.zback.json",
  "supportsAEngineIntegration": true
}
```

---

## 5. Required Properties

### 5.1 id

- Type: string
- Required: yes
- Meaning: unique profile identifier

### 5.2 version

- Type: string
- Required: yes
- Meaning: semantic version of the profile package

### 5.3 displayName

- Type: string
- Required: yes
- Meaning: user-facing profile name

### 5.4 toolchainExecutable

- Type: string
- Required: yes
- Meaning: command or executable name used as profile entrypoint

### 5.5 compilerExecutable

- Type: string
- Required: yes
- Meaning: command or executable name of bound compiler backend

### 5.6 builtInModules

- Type: string array
- Required: yes
- Meaning: built-in module ids loaded by default for this profile

### 5.7 projectFileName

- Type: string
- Required: yes
- Meaning: canonical project file name for projects using this profile

### 5.8 supportsAEngineIntegration

- Type: boolean
- Required: yes
- Meaning: indicates whether profile includes dedicated AEngine integration package

---

## 6. Required C# Type

```csharp
namespace AEngine.ProjectProfiles;

public sealed class ProfileManifest
{
    public required string Id { get; init; }
    public required string Version { get; init; }
    public required string DisplayName { get; init; }
    public required string ToolchainExecutable { get; init; }
    public required string CompilerExecutable { get; init; }
    public required IReadOnlyList<string> BuiltInModules { get; init; }
    public required string ProjectFileName { get; init; }
    public required bool SupportsAEngineIntegration { get; init; }
}
```

---

## 7. Validation Rules

A valid profile manifest MUST satisfy all of the following:

1. `Id` is non-empty.
2. `Version` is non-empty.
3. `DisplayName` is non-empty.
4. `ToolchainExecutable` is non-empty.
5. `CompilerExecutable` is non-empty.
6. `ProjectFileName` is non-empty.
7. `BuiltInModules` is not null.
8. `BuiltInModules` contains unique module ids only.

---

## 8. Required Loader Type

```csharp
namespace AEngine.ProjectProfiles;

public sealed class ProfileManifestLoader
{
    public ProfileManifest Load(string manifestPath)
    {
        ArgumentNullException.ThrowIfNull(manifestPath);
        // implementation required
    }
}
```

---

## 9. Required Loader Behavior

`Load()` MUST:

1. validate path argument;
2. check that file exists;
3. read UTF-8 JSON;
4. deserialize into `ProfileManifest`;
5. validate all required properties;
6. return manifest instance.

If file does not exist, `Load()` MUST throw `FileNotFoundException`.
If JSON is invalid, `Load()` MUST throw `InvalidDataException`.
If schema is invalid, `Load()` MUST throw `InvalidDataException`.

---

## 10. Minimal Required Skeleton

```csharp
namespace AEngine.ProjectProfiles;

using System.Text.Json;

public sealed class ProfileManifestLoader
{
    public ProfileManifest Load(string manifestPath)
    {
        ArgumentNullException.ThrowIfNull(manifestPath);

        if (!File.Exists(manifestPath))
            throw new FileNotFoundException("Profile manifest not found.", manifestPath);

        var json = File.ReadAllText(manifestPath);
        var manifest = JsonSerializer.Deserialize<ProfileManifest>(json);

        if (manifest is null)
            throw new InvalidDataException("Profile manifest could not be deserialized.");

        if (string.IsNullOrWhiteSpace(manifest.Id))
            throw new InvalidDataException("Profile manifest Id is required.");

        if (manifest.BuiltInModules is null)
            throw new InvalidDataException("Profile manifest BuiltInModules is required.");

        return manifest;
    }
}
```

Any materially different manifest contract is non-compliant.
