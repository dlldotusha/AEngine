# State Machines — Implementation Specification

## 1. Purpose

Defines formal state machines for key runtime components.

---

## 2. Execution Context State Machine

States:
- Created
- Initialized
- Running
- ShuttingDown
- Terminated

Transitions:

Created → Initialized
Initialized → Running
Running → ShuttingDown
ShuttingDown → Terminated

---

## 3. Required Rules

- Initialize() moves Created → Initialized
- First Tick() moves Initialized → Running
- Shutdown() moves Running → ShuttingDown
- After cleanup → Terminated

---

## 4. Connection State Machine

States:
- Accepted
- Active
- Closed

Transitions:

Accepted → Active
Active → Closed

---

## 5. Build Pipeline State Machine

States:
- Idle
- Validating
- Preparing
- Compiling
- Finalizing
- Completed
- Failed

Transitions:

Idle → Validating → Preparing → Compiling → Finalizing → Completed

Any step → Failed on error

---

## 6. Plugin State Machine

States:
- Discovered
- Loaded
- Initialized
- Faulted

---

## 7. Module State Machine

States:
- Registered
- Initialized
- Active
- Disposed
