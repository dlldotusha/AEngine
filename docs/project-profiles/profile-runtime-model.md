# Project Profile Runtime Model — Code-Level Specification

## 1. Purpose

Defines how a Project Profile models runtime execution semantics.

This specification removes ambiguity: profiles must define how built artifacts are executed.

---

## 2. Required Concept: Execution Context

A Project Profile MUST define an **Execution Context**.

Execution Context = isolated runtime instance of project logic.

Examples:
- one player session;
- one server instance;
- one simulation instance.

---

## 3. Default Model (ZBack baseline)

The default model MUST be:

- each connection = new execution context;
- contexts are isolated by default;
- no shared memory between contexts unless module explicitly enables it.

This means:

- system behaves like launching a game instance per connection;
- NOT multiplayer by default;
- identical connections produce identical independent worlds.

---

## 4. Required Interfaces

Profiles MUST define runtime contract via these types.

### 4.1 ExecutionContextId

```csharp
public readonly struct ExecutionContextId
{
    public required string Value { get; init; }
}
```

---

### 4.2 IExecutionContext

```csharp
public interface IExecutionContext
{
    ExecutionContextId Id { get; }
    void Initialize();
    void Tick();
    void Shutdown();
}
```

Rules:
- Initialize MUST be called once
- Tick MAY be called repeatedly
- Shutdown MUST be idempotent

---

### 4.3 IExecutionContextFactory

```csharp
public interface IExecutionContextFactory
{
    IExecutionContext CreateNew();
}
```

---

## 5. Connection Model (ZBack)

ZBack runtime MUST follow:

```text
on client connect:
    ctx = factory.CreateNew()
    ctx.Initialize()
    while connected:
        ctx.Tick()
    ctx.Shutdown()
```

No shared state is allowed by default.

---

## 6. Multiplayer Rule

Multiplayer MUST NOT exist implicitly.

Multiplayer requires:
- explicit module
- explicit shared-state model

---

## 7. Determinism

Execution context behavior MUST be deterministic given identical inputs.

---

## 8. Prohibited Behavior

Profiles MUST NOT:
- assume global world state exists;
- assume time system exists;
- assume player inventory model exists;
- assume networking model beyond connection abstraction.

All of these must be provided by modules.
