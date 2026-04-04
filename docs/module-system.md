# Module System Specification

## 1. Purpose

Modules are compile-time extensions of a Project Profile.
They define additional capabilities, APIs, and models available to a project.

Modules are NOT plugins and are NOT sandboxed.
They are trusted components of the profile.

---

## 2. Core Properties

Each module MUST define:

- id
- version
- supportedProfiles
- dependencies
- conflicts
- entry point

Example manifest:

```json
{
  "id": "session-module",
  "version": "1.0.0",
  "supportedProfiles": ["ZBack"],
  "dependencies": [],
  "conflicts": []
}
```

---

## 3. Lifecycle

Module lifecycle:

1. discovery
2. validation
3. dependency resolution
4. initialization
5. registration of hooks

---

## 4. Dependency Rules

- dependencies MUST be acyclic
- missing dependency MUST fail build
- version mismatches MUST fail build

---

## 5. Conflict Rules

If two modules declare conflict:
- build MUST fail

No automatic resolution is allowed.

---

## 6. Hook System

Modules may register hooks into ZBack pipeline:

- beforeCompile
- afterCompile
- validateProject

Hooks must be pure functions or side-effect controlled.

---

## 7. Isolation Rules

Modules:
- may interact with other modules only via declared contracts
- must not mutate global state unpredictably

---

## 8. Ordering

Modules must be initialized in topological order based on dependencies.

---

## 9. Compatibility

Modules MUST declare compatible profiles.
If mismatch:
- module must not load

---

## 10. Non-goals

Modules do NOT:
- provide UI directly (handled by plugins or profile integration)
- bypass compiler
- modify ACore
