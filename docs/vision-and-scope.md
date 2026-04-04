# Vision and Scope

## 1. Product Definition

AEngine is a platform for authoring, building, running, and debugging projects defined by Project Profiles.

AEngine is NOT a traditional game engine runtime in the Unity or Unreal sense.
It is a host environment plus project-specific toolchains.

The system is composed of:
- ACore — immutable platform foundation;
- Project Profiles — project-core definitions;
- Compilers — build backends;
- Modules — compile-time project extensions;
- Plugins — sandboxed environment extensions.

This separation is intentionally aligned with layered architecture principles found in industrial engine design, where tools, runtime, platform abstraction, and game-specific logic are separated rather than collapsed into a monolith. fileciteturn3file0

---

## 2. Core Design Goals

1. Keep ACore minimal and stable.
2. Move project-specific behavior into Project Profiles.
3. Treat compilers as independent external build backends.
4. Treat modules as trusted compile-time extensions.
5. Treat plugins as sandboxed developer tooling extensions.
6. Permit standalone CLI workflows with no GUI dependency.
7. Make architecture explicit enough that implementation can be generated from specification.

---

## 3. Non-Goals

The first implementation of AEngine does NOT aim to provide:
- a built-in gameplay runtime;
- a built-in world model;
- a built-in time system;
- a built-in networking stack;
- a built-in ECS;
- generic simulation systems such as water, physics, lighting;
- a scripting VM;
- a universal intermediate language owned by ACore.

These may appear later as Project Profile built-ins or modules, but not as assumptions of ACore.

---

## 4. Architectural Boundaries

### 4.1 ACore

ACore is immutable host infrastructure.
It may be extended only through explicitly supported plugin and profile integration surfaces.

### 4.2 Project Profile

A Project Profile defines the shape of a project.
It contains:
- one compiler contract binding;
- one built-in module set;
- project rules;
- project templates;
- orchestration behavior.

### 4.3 Compiler

A compiler only compiles.
It does not own project lifecycle commands such as `new` or `run`.
Those belong to the Project Profile toolchain.

### 4.4 Module

A module is trusted compile-time project functionality.
It is part of a profile or a profile-compatible extension set.

### 4.5 Plugin

A plugin is a sandboxed extension to the developer environment.
It does not alter compiled output or project semantics.

---

## 5. Product Editions

At the product level, AEngine is expected to have editions such as:
- Community;
- Personal;
- Team.

Differences between editions are primarily expected to be expressed through:
- included Project Profiles;
- included built-in modules;
- collaboration features.

Edition licensing is explicitly outside the scope of code architecture, except where feature flags or availability gates are required.

---

## 6. Terminology

- **ACore**: platform foundation.
- **Project Profile**: project-core definition and toolchain entrypoint.
- **Compiler**: external backend that transforms source to artifacts.
- **Module**: trusted compile-time project extension.
- **Plugin**: sandboxed environment extension.
- **Artifact**: any output produced by build.
- **Manifest**: machine-readable component metadata.
- **Diagnostics**: normalized warnings/errors/info produced by any subsystem.

---

## 7. Implementation Requirement

All architectural documents in `docs/` are normative for implementation.
If code and documentation diverge, documentation must be treated as the source of truth until the divergence is intentionally resolved.