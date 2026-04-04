# Canonical Schemas — Implementation Specification

## 1. Purpose

This document defines canonical schemas used across AEngine, Project Profiles, ZBack, modules, plugins, and compiler integration.

All JSON-producing and JSON-consuming components MUST use these schemas or exact supersets that preserve the required fields unchanged.

---

## 2. Canonical Diagnostic Entry Schema

```json
{
  "severity": "error",
  "code": "AE0001",
  "message": "Example message.",
  "source": "ZBack",
  "file": "src/main.zb",
  "range": {
    "startLine": 10,
    "startColumn": 5,
    "endLine": 10,
    "endColumn": 12
  },
  "timestampUtc": "2026-04-05T12:00:00Z"
}
```

Required fields:
- severity
- code
- message
- source
- timestampUtc

Optional fields:
- file
- range

---

## 3. Canonical Range Schema

```json
{
  "startLine": 1,
  "startColumn": 1,
  "endLine": 1,
  "endColumn": 1
}
```

All values MUST be 1-based integers.

---

## 4. Canonical Compiler Result Schema

```json
{
  "compilerId": "zcompiler",
  "compilerVersion": "1.0.0",
  "success": true,
  "diagnostics": [],
  "artifacts": [
    {
      "kind": "binary",
      "path": "out/zserver.bin"
    }
  ]
}
```

Required fields:
- compilerId
- compilerVersion
- success
- diagnostics
- artifacts

---

## 5. Canonical Artifact Schema

```json
{
  "kind": "binary",
  "path": "out/zserver.bin"
}
```

Required fields:
- kind
- path

---

## 6. Canonical Module Manifest Schema

```json
{
  "id": "session",
  "version": "1.0.0",
  "displayName": "Session Module",
  "entryType": "ModuleEntry",
  "dependencies": [],
  "conflicts": []
}
```

Required fields:
- id
- version
- displayName
- entryType
- dependencies
- conflicts

---

## 7. Canonical Plugin Manifest Schema

```json
{
  "id": "color-preview",
  "version": "1.0.0",
  "displayName": "Color Preview",
  "entryType": "PluginEntry",
  "permissionsRequested": ["UI.Panel.Create"]
}
```

Required fields:
- id
- version
- displayName
- entryType
- permissionsRequested

---

## 8. Canonical Project File Schema

```json
{
  "profile": "ZBack",
  "profileVersion": "1.0.0",
  "name": "MyProject",
  "sources": ["src/"],
  "modules": ["session", "tick"],
  "build": {
    "configuration": "debug"
  },
  "output": {
    "path": "out/"
  }
}
```

Required fields:
- profile
- profileVersion
- name
- sources
- modules
- build
- output

---

## 9. Canonical Build Metadata Schema

```json
{
  "toolchainId": "zback",
  "profileId": "ZBack",
  "compilerId": "zcompiler",
  "compilerVersion": "1.0.0",
  "artifacts": [
    {
      "kind": "binary",
      "path": "out/zserver.bin"
    }
  ],
  "generatedAtUtc": "2026-04-05T12:00:00Z"
}
```

Required fields:
- toolchainId
- profileId
- compilerId
- compilerVersion
- artifacts
- generatedAtUtc

---

## 10. Rule

Whenever a component writes one of these structures to disk or stdout, field names and required field meanings MUST remain identical.
