# ACore — Technical Specification

## 1. Purpose

ACore is the host environment responsible for:

- loading project profiles;
- managing plugins;
- providing UI shell;
- enforcing security boundaries;
- aggregating diagnostics.

ACore MUST NOT:

- perform builds;
- contain compiler logic;
- contain project-specific rules;
- execute runtime pipelines.

---

## 2. Runtime Model

ACore is a long-lived process.

It exposes internal subsystems through interfaces:

- ProfileRegistry
- PluginRegistry
- PermissionManager
- UIShell
- DiagnosticsHub

All subsystems MUST be initialized in deterministic order.

---

## 3. Subsystems

### 3.1 ProfileRegistry

Responsibilities:

- discover profiles;
- load profile manifests;
- validate compatibility;
- register profile runtime adapter.

Interface:

```
interface ProfileRegistry {
  loadProfiles(path: string): Profile[]
  getProfile(id: string): Profile
}
```

---

### 3.2 PluginRegistry

Responsibilities:

- discover plugins;
- load manifests;
- resolve dependencies;
- initialize plugins.

Lifecycle:

1. discover
2. validate
3. permission check
4. initialize

---

### 3.3 PermissionManager

Responsibilities:

- evaluate plugin permissions;
- provide capability tokens;
- enforce access boundaries.

Permissions are evaluated BEFORE plugin initialization.

---

### 3.4 UIShell

Responsibilities:

- window management;
- panels;
- docking;
- integration points for plugins.

Constraints:

- UI must not depend on compiler;
- UI must communicate via interfaces only.

---

### 3.5 DiagnosticsHub

Responsibilities:

- collect diagnostics from all subsystems;
- normalize format;
- expose stream to UI.

Diagnostic format:

```
{
  severity: "error" | "warning" | "info",
  message: string,
  code: string,
  source: string
}
```

---

## 4. Initialization Order

1. DiagnosticsHub
2. PermissionManager
3. PluginRegistry
4. ProfileRegistry
5. UIShell

---

## 5. Extension Points

ACore exposes extension points ONLY via:

- plugin API
- UI hooks
- diagnostics stream

Direct access to internal state is forbidden.

---

## 6. Failure Model

ACore must:

- isolate plugin crashes;
- continue running if plugin fails;
- surface all failures via DiagnosticsHub.

---

## 7. Threading Model (minimal requirement)

- main thread: UI
- worker threads: plugin execution

No subsystem may block UI thread.

---

## 8. Non-goals

- no ECS
- no rendering
- no physics
- no runtime simulation

ACore is a tool host, not an engine runtime.
