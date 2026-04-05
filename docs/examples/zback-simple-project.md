# ZBack Simple Project — End-to-End Example

## 1. Purpose

This document demonstrates the complete lifecycle of a ZBack project from project file to runtime execution.

---

## 2. Project File

File: `project.zback.json`

```json
{
  "profile": "ZBack",
  "profileVersion": "1.0.0",
  "name": "ExampleProject",
  "sources": ["src/"],
  "modules": [
    "session",
    "tick",
    "input",
    "execution-context"
  ],
  "build": {
    "configuration": "debug"
  },
  "output": {
    "path": "out/"
  }
}
```

---

## 3. Build Command

Command:

```text
zback build
```

---

## 4. Build Pipeline Execution

Steps executed:

1. Load project file.
2. Validate profile and version compatibility.
3. Load ZBack profile manifest.
4. Validate module compatibility.
5. Resolve compiler executable (zcompiler).
6. Invoke compiler.

---

## 5. Compiler Output

Example output:

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

---

## 6. Runtime Launch

Command:

```text
zback run
```

---

## 7. Runtime Execution Flow

### Step 1: Runtime startup

- load build metadata
- validate compatibility
- create runtime host

---

### Step 2: Connection accepted

- ConnectionModule returns connection

---

### Step 3: Execution context created

- new execution context instance created

---

### Step 4: Session created

- SessionModule.CreateSession(contextId)

---

### Step 5: Initialize

- ExecutionContextModule.Initialize(context)

---

### Step 6: Tick loop

Repeated until connection closes:

1. InputModule captures input
2. TickModule advances tick
3. ExecutionContextModule.Tick(context)

---

### Step 7: Shutdown

- ExecutionContextModule.Shutdown(context)
- SessionModule.DestroySession(contextId)
- connection disposed

---

## 8. Result

Each connection results in:

- one execution context
- one session
- one isolated runtime instance

No interaction occurs between contexts unless multiplayer modules are explicitly added.

---

## 9. Mental Model

The runtime behaves as:

"server-side singleplayer per connection"

This is the default ZServer model.
