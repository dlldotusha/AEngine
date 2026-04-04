# Execution Pipeline Specification

## 1. Purpose

Defines the exact steps executed during `zback build` and `zback run`.

---

## 2. Build Pipeline

Ordered steps:

1. Load project file
2. Validate schema
3. Resolve profile
4. Load modules
5. Resolve dependencies
6. Execute module validation hooks
7. Invoke compiler
8. Collect artifacts
9. Emit diagnostics

---

## 3. Run Pipeline

1. Ensure build exists
2. Load runtime metadata
3. Launch runtime process
4. Attach diagnostics stream

---

## 4. Hook Integration

Modules may inject:

- pre-validation
- post-validation
- pre-compile
- post-compile

---

## 5. Failure Rules

- any failure aborts pipeline
- diagnostics must be printed

---

## 6. Determinism

Build must be deterministic given same inputs.
