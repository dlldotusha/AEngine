# Versioning and Compatibility — Production Specification

## 1. Purpose

This document defines versioning rules and compatibility rules for ACore, Project Profiles, compilers, modules, plugins, project files, and build artifacts.

This document is normative.

---

## 2. Versioning Format

All versioned components MUST use semantic versioning in the form:

```text
MAJOR.MINOR.PATCH
```

Examples:
- `1.0.0`
- `2026.1.0`

The implementation MAY choose calendar-based major versions, but the three-part format MUST remain intact.

---

## 3. Semantic Meaning

### 3.1 MAJOR

Increment MAJOR when a change is not backward compatible.

### 3.2 MINOR

Increment MINOR when a backward-compatible capability is added.

### 3.3 PATCH

Increment PATCH when a backward-compatible fix is made without adding a new capability.

---

## 4. Compatibility Matrix

### 4.1 ACore ↔ Project Profile

A profile MUST declare the minimum ACore version it supports.

Required manifest field:

```json
"minimumACoreVersion": "1.0.0"
```

ACore MUST refuse to load a profile if:
- ACore version is lower than `minimumACoreVersion`.

### 4.2 Project Profile ↔ Compiler

A profile MUST declare the compiler id and allowed compiler version range.

Required fields:

```json
"compilerExecutable": "zcompiler",
"compilerVersionRange": ">=1.0.0 <2.0.0"
```

The toolchain MUST refuse compiler invocation if the compiler version does not satisfy the declared range.

### 4.3 Project Profile ↔ Module

A module MUST declare supported profile ids and supported profile version range.

Required fields:

```json
"supportedProfiles": ["ZBack"],
"supportedProfileVersionRange": ">=1.0.0 <2.0.0"
```

The profile MUST refuse to load a module that does not satisfy both profile id and version compatibility.

### 4.4 Plugin ↔ ACore

A plugin MUST declare the minimum ACore version it supports.

Required field:

```json
"minimumACoreVersion": "1.0.0"
```

ACore MUST refuse initialization of incompatible plugins.

### 4.5 Project File ↔ Profile

The project file MUST declare:

```json
"profile": "ZBack",
"profileVersion": "1.0.0"
```

ZBack MUST refuse to load a project file if the project declares a profile version that is incompatible with the installed profile package.

---

## 5. Build Artifact Compatibility

Every build output MUST emit build metadata containing:

- toolchain id
- profile id
- profile version
- compiler id
- compiler version
- generatedAtUtc

Any runtime attempting to launch artifacts MUST validate this metadata before launch.

---

## 6. Compatibility Evaluation Rules

When a version range is evaluated, the implementation MUST use inclusive lower bound and exclusive upper bound semantics.

Example:

```text
>=1.0.0 <2.0.0
```

means:
- `1.0.0` allowed
- `1.4.7` allowed
- `2.0.0` not allowed

---

## 7. Failure Rules

If compatibility validation fails, the system MUST:

1. emit a diagnostic;
2. stop loading that component;
3. continue only if the failed component is optional.

If the failed component is required, startup or build MUST fail.

---

## 8. Required Example Fields

### Profile manifest

```json
{
  "id": "ZBack",
  "version": "1.0.0",
  "minimumACoreVersion": "1.0.0",
  "compilerExecutable": "zcompiler",
  "compilerVersionRange": ">=1.0.0 <2.0.0"
}
```

### Module manifest

```json
{
  "id": "session",
  "version": "1.0.0",
  "supportedProfiles": ["ZBack"],
  "supportedProfileVersionRange": ">=1.0.0 <2.0.0"
}
```

### Plugin manifest

```json
{
  "id": "color-preview",
  "version": "1.0.0",
  "minimumACoreVersion": "1.0.0"
}
```

---

## 9. Production Rule

No component may be loaded, invoked, or executed unless compatibility has been validated first.
