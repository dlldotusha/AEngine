# ZBack Built-in Modules — Implementation Specification

## 1. Purpose

This document defines the required built-in modules that MUST be included with the ZBack profile.

These modules provide baseline functionality required for Minecraft-oriented server projects.

These modules are NOT part of ACore.
They are part of the ZBack profile.

---

## 2. Required Modules

The first implementation of ZBack MUST include the following modules:

- connection
- session

---

## 3. Connection Module

### 3.1 Responsibility

The connection module is responsible for:

- accepting incoming connections;
- creating execution contexts;
- binding connection lifecycle to execution lifecycle.

---

### 3.2 Required Behavior

On new connection:

```text
ctx = executionContextFactory.CreateNew()
ctx.Initialize()

while connection is active:
    ctx.Tick()

ctx.Shutdown()
```

---

### 3.3 Output

The module MUST provide a connection abstraction that can be used by higher-level modules.

---

## 4. Session Module

### 4.1 Responsibility

The session module is responsible for:

- maintaining per-connection session state;
- exposing session-scoped storage;
- associating data with execution context.

---

### 4.2 Required Behavior

The session module MUST:

- create a session object during context initialization;
- attach session object to execution context;
- destroy session object during context shutdown.

---

## 5. Module Initialization Order

The modules MUST be initialized in this order:

1. connection
2. session

---

## 6. Integration with Execution Context

Modules MUST use the execution context system defined by the profile runtime model.

They MUST NOT create alternative execution loops.

---

## 7. Extensibility

Additional modules MAY be added to provide:

- world systems;
- inventory systems;
- time systems;
- networking extensions;

These are not part of the base ZBack profile.

---

## 8. Required Guarantee

After loading built-in modules, the system MUST be capable of:

- accepting a connection;
- creating an execution context;
- running a Tick loop;
- shutting down cleanly.

No additional modules are required to reach this baseline behavior.
