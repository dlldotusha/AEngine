# Plugin System Specification

## 1. Purpose

Plugins extend ACore and developer experience.
They do NOT affect compiled output.

---

## 2. Core Rules

Plugins:
- run inside sandbox
- require permissions
- cannot access internal systems directly

---

## 3. Manifest

```json
{
  "id": "plugin-id",
  "version": "1.0.0",
  "permissionsRequested": []
}
```

---

## 4. Permission Model

Permissions are granted per:

plugin id + version

NOT by signature.

---

## 5. Allowed Capabilities

Examples:
- UI.Panel.Create
- Inspector.Register
- Metadata.Read

---

## 6. Enforcement

ACore must:
- check permissions before execution
- deny unauthorized calls

---

## 7. Isolation

Plugins must not:
- crash ACore
- access filesystem freely
- execute arbitrary system commands

---

## 8. Failure Handling

If plugin fails:
- isolate
- log
- continue system

---

## 9. Non-goals

Plugins do NOT:
- affect compiler
- affect modules
- define project semantics
