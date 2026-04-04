# Project Format Specification

## 1. Purpose

Defines the structure of a project that can be consumed by ZBack and associated compilers.

---

## 2. Project Root

A project MUST contain a root descriptor file:

```
project.zback.json
```

---

## 3. Project File Schema

```json
{
  "profile": "ZBack",
  "profileVersion": "1.0.0",
  "name": "ProjectName",
  "sources": ["src/"],
  "modules": ["connection", "session"],
  "build": {
    "configuration": "debug"
  },
  "output": {
    "path": "out/"
  }
}
```

---

## 4. Directory Layout

```
/project
  project.zback.json
  /src
  /out
  /modules (optional)
```

---

## 5. Rules

- profile MUST exist
- modules MUST be compatible
- source paths MUST exist

---

## 6. Validation

ZBack MUST validate:
- schema correctness
- dependency graph
- module compatibility

Failure MUST stop build.
