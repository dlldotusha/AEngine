# Failure Semantics — Production Specification

## 1. Purpose

Defines how failures must be handled across all components.

---

## 2. Failure Categories

Failures are categorized as:

- Validation failure
- Runtime failure
- Module failure
- Plugin failure
- Compiler failure
- Environment failure

---

## 3. Execution Context Failure

If any module throws during Tick:

The runtime MUST:

1. stop the current execution context;
2. call Shutdown(context);
3. emit a diagnostic entry;
4. dispose the connection;
5. continue accepting new connections.

---

## 4. Module Failure

If a module fails during Initialize:

The runtime MUST:

1. mark the module as failed;
2. emit a diagnostic;
3. abort context initialization;
4. dispose the context;
5. continue processing other connections.

---

## 5. Plugin Failure

If a plugin throws:

The system MUST:

1. catch the exception at the plugin boundary;
2. emit a diagnostic;
3. mark plugin as Faulted;
4. continue running ACore.

---

## 6. Compiler Failure

If compiler returns failure:

The toolchain MUST:

1. stop the build pipeline;
2. emit diagnostics;
3. not produce runtime artifacts;
4. return non-zero exit code.

---

## 7. Validation Failure

If validation fails:

The system MUST:

1. stop processing immediately;
2. emit diagnostics;
3. not proceed to compilation or runtime.

---

## 8. Environment Failure

Examples:
- missing compiler
- inaccessible directory

The system MUST:

1. emit diagnostic;
2. abort current operation;
3. return failure exit code.

---

## 9. Required Rule

No failure may propagate uncaught across system boundaries.
All failures must be translated into diagnostics and controlled shutdown behavior.
