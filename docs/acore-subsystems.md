# ACore Subsystems — Implementation Specification

## 1. Purpose

This document specifies the internal subsystem model of ACore in enough detail that it can be implemented directly.

This document is normative.

---

## 2. Required Subsystems

ACore MUST contain the following first-class subsystems:

1. DiagnosticsHub
2. ConfigurationService
3. PermissionManager
4. PluginRegistry
5. ProfileRegistry
6. UIShell

No additional subsystem may be introduced into the bootstrap sequence unless it is documented in this package.

---

## 3. Shared Rules

All subsystems MUST follow these rules:

- implement the common lifecycle interface;
- accept dependencies through constructor injection only;
- never instantiate peer subsystems directly;
- emit diagnostics via DiagnosticsHub only;
- avoid static mutable global state;
- expose explicit public interfaces.

Required interface:

```csharp
public interface ISubsystem
{
    string Id { get; }
    SubsystemState State { get; }
    void Initialize();
    void Start();
    void Stop();
}
```

Required enum:

```csharp
public enum SubsystemState
{
    Created,
    Initialized,
    Running,
    Stopped,
    Faulted
}
```

---

## 4. DiagnosticsHub

### 4.1 Responsibility

DiagnosticsHub is the single sink and broker for all diagnostics inside ACore.

### 4.2 Required API

```csharp
public interface IDiagnosticsHub : ISubsystem
{
    void Publish(DiagnosticEntry entry);
    IReadOnlyList<DiagnosticEntry> Snapshot();
    event EventHandler<DiagnosticEntry> DiagnosticPublished;
}
```

### 4.3 Rules

- MUST be thread-safe.
- MUST preserve diagnostic ordering per publisher thread.
- MUST timestamp diagnostics.
- MUST support snapshot retrieval for UI.

### 4.4 Storage

First implementation MUST keep diagnostics in memory ring buffer.

Required configuration:
- max entry count;
- overflow policy = drop oldest.

---

## 5. ConfigurationService

### 5.1 Responsibility

Loads and exposes ACore configuration.

### 5.2 Required API

```csharp
public interface IConfigurationService : ISubsystem
{
    T GetSection<T>(string sectionName) where T : class, new();
    bool TryGetSection<T>(string sectionName, out T value) where T : class, new();
}
```

### 5.3 Rules

- MUST load configuration before other services request values.
- MUST validate configuration structure.
- MUST emit diagnostics on invalid entries.
- MUST fail startup only on critical config corruption.

---

## 6. PermissionManager

### 6.1 Responsibility

Resolves and enforces plugin permissions.

### 6.2 Required API

```csharp
public interface IPermissionManager : ISubsystem
{
    PermissionGrantSet Resolve(string pluginId, string pluginVersion);
    bool IsAllowed(string pluginId, string pluginVersion, string capability);
}
```

### 6.3 Rules

- MUST be queried before plugin initialization.
- MUST support local fallback policy store.
- MUST deny by default.
- MUST not infer permissions from signatures.

---

## 7. PluginRegistry

### 7.1 Responsibility

Discovers, validates, loads, and tracks plugins.

### 7.2 Required API

```csharp
public interface IPluginRegistry : ISubsystem
{
    IReadOnlyList<PluginDescriptor> LoadedPlugins { get; }
    void Discover(string rootPath);
    void LoadAll();
}
```

### 7.3 Discovery Algorithm

1. Enumerate plugin directories.
2. Read `plugin.json`.
3. Validate schema.
4. Query permissions.
5. Create descriptor.

### 7.4 Loading Rules

- Plugins MUST load only after successful permission resolution.
- One plugin failure MUST NOT abort ACore startup.
- Failed plugins MUST be marked Faulted.

---

## 8. ProfileRegistry

### 8.1 Responsibility

Discovers and registers Project Profiles.

### 8.2 Required API

```csharp
public interface IProfileRegistry : ISubsystem
{
    IReadOnlyList<ProfileDescriptor> Profiles { get; }
    void Discover(string rootPath);
    ProfileDescriptor GetRequired(string id);
}
```

### 8.3 Rules

- MUST validate manifests before registration.
- MUST fail startup if no valid profiles are available.
- MUST support one default profile selection strategy.

---

## 9. UIShell

### 9.1 Responsibility

Hosts the visible application shell.

### 9.2 Required API

```csharp
public interface IUiShell : ISubsystem
{
    void RegisterPanel(UiPanelDescriptor panel);
    void ShowMainWindow();
}
```

### 9.3 Rules

- MUST remain implementation-neutral in docs.
- MUST support panel registration from plugins and profile integrations.
- MUST subscribe to DiagnosticsHub.

---

## 10. Dependency Graph

Required dependency graph:

- DiagnosticsHub: no dependencies
- ConfigurationService: depends on DiagnosticsHub
- PermissionManager: depends on DiagnosticsHub, ConfigurationService
- PluginRegistry: depends on DiagnosticsHub, PermissionManager
- ProfileRegistry: depends on DiagnosticsHub, ConfigurationService
- UIShell: depends on DiagnosticsHub, ProfileRegistry, PluginRegistry

No cycles are allowed.

---

## 11. Factory Rule

ACore MUST use an explicit composition root.

It MUST NOT use:
- reflection-based auto-wiring;
- implicit assembly scanning for subsystem construction;
- service locator hidden inside subsystems.

---

## 12. Failure Policy

Subsystem failure categories:

- Recoverable: plugin load failure, UI panel registration failure
- Non-recoverable: DiagnosticsHub init failure, ProfileRegistry total failure

ACore MUST enter Degraded state on recoverable failures.
ACore MUST abort startup on non-recoverable failures.
