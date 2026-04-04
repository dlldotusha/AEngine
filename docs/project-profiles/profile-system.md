# Project Profile System — Detailed Specification

## 1. Purpose

The Project Profile system defines how a project behaves, builds, and runs.

A profile is NOT optional. Every project MUST have exactly one active profile.

---

## 2. Responsibilities

A profile is responsible for:

- defining project structure rules;
- defining build orchestration behavior;
- defining runtime execution semantics;
- declaring built-in modules;
- binding a compiler;
- defining CLI entrypoints.

---

## 3. Profile Composition

A profile is composed of:

1. Profile Manifest
2. Toolchain (CLI implementation, e.g., ZBack)
3. Compiler binding
4. Built-in modules
5. Integration layer for ACore

---

## 4. Profile Manifest (Strict Requirements)

Example:

```json
{
  "id": "ZBack",
  "version": "1.0.0",
  "compiler": "zcompiler",
  "modules": ["connection", "session"],
  "entry": "zback"
}
```

Rules:

- `id` must be unique;
- `compiler` must exist;
- modules must be compatible;
- entry must be executable.

---

## 5. Profile Lifecycle

### 5.1 Load

Steps:

1. Read manifest
2. Validate schema
3. Validate compiler existence
4. Validate module compatibility

---

### 5.2 Initialize

1. Load modules
2. Resolve dependencies
3. Initialize modules

---

### 5.3 Execute

Profile exposes commands such as:

- build
- run
- validate

Each command maps to a pipeline.

---

## 6. Compatibility Model

Profiles must explicitly declare compatibility with:

- modules
- compiler versions

If mismatch:
- hard failure

---

## 7. Isolation

Profiles must not:

- modify ACore internals
- bypass permission system

---

## 8. Extension

Profiles may be extended by:

- additional modules
- tooling plugins (indirectly)

---

## 9. Non-goals

Profiles do not:

- provide UI directly (delegated to integration layer)
- define plugin permissions
