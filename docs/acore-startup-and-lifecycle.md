# ACore Startup and Lifecycle — Implementation Specification

## 1. Purpose

Defines exact startup sequence, state transitions, and subsystem initialization rules for ACore.

This document is IMPLEMENTATION-NORMATIVE.

---

## 2. Global State Machine

ACore MUST implement the following states:

- NotStarted
- Initializing
- Running
- Degraded
- ShuttingDown
- Terminated

Transitions:

```
NotStarted → Initializing → Running
Running → Degraded (on recoverable subsystem failure)
Running → ShuttingDown → Terminated
```

---

## 3. Startup Sequence (STRICT ORDER)

ACore MUST initialize subsystems in the following exact order:

1. DiagnosticsHub
2. ConfigurationService
3. PermissionManager
4. PluginRegistry
5. ProfileRegistry
6. UIShell

If any subsystem fails:

- emit diagnostic
- attempt recovery if possible
- otherwise abort startup

---

## 4. Subsystem Initialization Contract

Each subsystem MUST implement:

```
interface ISubsystem {
    void Initialize(ACoreContext context);
    void Start();
    void Stop();
}
```

Rules:
- Initialize MUST NOT perform long blocking work
- Start MAY perform async work
- Stop MUST be idempotent

---

## 5. Context Object

ACore MUST construct a single shared context:

```
class ACoreContext {
    DiagnosticsHub Diagnostics;
    ConfigurationService Configuration;
    PermissionManager Permissions;
}
```

No subsystem may directly construct other subsystems.

---

## 6. Failure Handling Rules

If PluginRegistry fails:
- continue startup
- mark system as Degraded

If ProfileRegistry fails:
- startup MUST fail

If UIShell fails:
- fallback to headless mode

---

## 7. Shutdown Sequence

Reverse order:

1. UIShell
2. ProfileRegistry
3. PluginRegistry
4. PermissionManager
5. ConfigurationService
6. DiagnosticsHub

All Stop() calls MUST be safe to call multiple times.

---

## 8. Threading Rules

- UI thread MUST remain responsive
- long work MUST be offloaded
- DiagnosticsHub must be thread-safe

---

## 9. Logging Requirements

Each state transition MUST emit a log entry.
Each subsystem start/stop MUST emit a log entry.

---

## 10. Prohibited Behaviors

- circular initialization dependencies
- blocking UI thread during Initialize()
- direct subsystem instantiation bypassing context
