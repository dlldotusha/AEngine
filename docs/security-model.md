# Security Model — Production Specification

## 1. Purpose

This document defines security boundaries and enforcement rules for AEngine, ACore, Project Profiles, modules, plugins, and runtime execution.

This document is normative.

---

## 2. Trust Levels

The first implementation MUST use exactly two trust levels.

### 2.1 Trusted

Trusted components are:
- ACore subsystems
- Project Profiles
- built-in modules
- explicitly installed profile modules
- compiler executables selected by profile configuration

Trusted components may access internal profile/runtime APIs directly.

### 2.2 Sandboxed

Sandboxed components are:
- plugins running inside ACore

Sandboxed components may access only APIs explicitly exposed through PluginContext.

---

## 3. Security Boundary Rules

### 3.1 ACore Boundary

ACore MUST expose only:
- subsystem interfaces;
- plugin APIs;
- profile integration interfaces.

ACore MUST NOT expose internal mutable subsystem state directly.

### 3.2 Profile Boundary

A Project Profile MUST expose:
- manifest metadata;
- toolchain entrypoint;
- built-in module list;
- runtime integration surface.

### 3.3 Module Boundary

Modules are trusted runtime components.
Modules MUST follow profile contracts and module lifecycle rules.
Modules are not sandboxed.

### 3.4 Plugin Boundary

Plugins MUST operate through PluginContext only.
A plugin MUST NOT receive direct references to ACore internal objects beyond the APIs defined in the plugin contract.

---

## 4. Capability Enforcement

Every privileged plugin action MUST be protected by a capability check.

Required enforcement rule:

1. Plugin requests action through PluginContext API.
2. API calls `Permissions.Demand(capability)`.
3. If allowed, action proceeds.
4. If denied, `PluginPermissionDeniedException` is thrown.
5. Boundary catches exception and emits diagnostic.

The capability check MUST happen inside the host API, not only inside plugin code.

---

## 5. Required Capability Names

The first implementation MUST support at minimum:
- `UI.Panel.Create`
- `UI.Command.Register`
- `Metadata.Read`
- `Diagnostics.Write`

Additional capabilities may be introduced later, but these names and meanings are fixed.

---

## 6. Permission Source Rules

Permissions MUST be resolved by:
- plugin id
- plugin version

Permissions MUST NOT be inferred from:
- code signing status
- filename
- installation directory

Default rule: deny unless explicitly granted.

---

## 7. Runtime Isolation Rules

### 7.1 ZServer Default Isolation

The default ZServer runtime MUST isolate execution contexts from each other.

This means:
- no implicit shared gameplay state;
- no implicit cross-context messaging;
- no implicit replicated world state.

### 7.2 Multiplayer Extension

Cross-context interaction may exist only when provided by explicit multiplayer-related modules.

---

## 8. Compiler Trust Rule

The compiler executable selected by a profile is treated as trusted for the purposes of build execution.

The toolchain MUST validate:
- compiler identity;
- compiler version compatibility;
- executable path existence.

If validation fails, build MUST abort.

---

## 9. Failure Handling at Security Boundaries

If a plugin attempts a disallowed action:

The host MUST:
1. deny the action;
2. emit a diagnostic;
3. keep ACore running;
4. preserve plugin fault isolation.

If a trusted module fails, normal runtime failure semantics apply.

---

## 10. Production Enforcement Rule

No security-sensitive rule may rely on plugin self-policing.
Enforcement MUST occur at the boundary controlled by the host or runtime.
