# ZBack — Technical Specification

## 1. Purpose

ZBack is a CLI orchestrator responsible for:

- project lifecycle management;
- invoking compiler processes;
- managing modules;
- executing runtime pipelines.

ZBack is NOT:

- a UI tool;
- a compiler;
- part of ACore runtime.

---

## 2. Execution Model

ZBack is a short-lived CLI process.

Each command:

1. loads project config;
2. resolves profile;
3. builds execution graph;
4. executes steps.

---

## 3. CLI Commands

### 3.1 new

Creates project structure.

### 3.2 build

Pipeline:

1. validate project;
2. load modules;
3. call compiler;
4. collect artifacts.

### 3.3 run

- ensures build exists;
- launches runtime.

### 3.4 validate

- schema validation;
- dependency resolution.

### 3.5 doctor

- environment diagnostics;
- dependency checks.

---

## 4. Compiler Contract

ZBack communicates with compiler via CLI.

Example:

```
zcompiler compile <project>
```

Compiler MUST:

- return exit code;
- output diagnostics JSON;
- write artifacts.

---

## 5. Module System Integration

ZBack loads modules defined in project:

```
modules: ["connection", "session"]
```

Each module:

- has lifecycle;
- can register hooks;
- may depend on other modules.

---

## 6. Execution Graph

ZBack builds DAG:

nodes:
- validation
- module init
- compile
- post-process

No cycles allowed.

---

## 7. Error Handling

ZBack must:

- stop on critical failure;
- print diagnostics;
- return non-zero exit code.

---

## 8. Output

Build produces:

- artifacts directory;
- diagnostics.json;
- metadata.json;

---

## 9. Environment Isolation

ZBack must not depend on ACore.

Standalone execution is mandatory.

---

## 10. Non-goals

- no UI
- no plugin system
- no long-running process
