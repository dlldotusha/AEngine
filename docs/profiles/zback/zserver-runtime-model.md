# ZServer Runtime Model — Implementation Specification

## 1. Purpose

This document defines the runtime model of the ZServer target used by ZBack.

ZServer is the default single-session runtime target for ZBack.
It represents a server-side binary to which a player connects, but each connection is treated as its own isolated runtime instance.

This document is normative.

---

## 2. Core Definition

ZServer MUST be treated as follows:

- one built server binary;
- one incoming connection creates one isolated execution context;
- that execution context behaves as that connection's personal game instance;
- no interaction between execution contexts exists by default.

In practical terms, ZServer acts like server-side singleplayer by default.

---

## 3. Required Default Behavior

On connection:

1. accept connection;
2. create new execution context;
3. initialize built-in modules for that context;
4. run tick loop for that context;
5. shut down context when connection ends.

The runtime MUST do this for every connection independently.

---

## 4. Shared-State Rule

Default ZServer runtime MUST NOT introduce any implicit shared state between different connections.

This means the runtime implementation must not:

- share gameplay state across contexts;
- share world state across contexts;
- replicate entities across contexts;
- deliver gameplay messages across contexts.

If shared state is required, it MUST be introduced by an explicit multiplayer-capable module.

---

## 5. Required Terms

### 5.1 Personal Instance

A personal instance is the isolated execution context created for a single connection.

### 5.2 Multiplayer Server

A multiplayer-capable runtime is NOT the default ZServer runtime.
It is a higher-level composition created by adding explicit multiplayer modules or by using a different runtime configuration.

---

## 6. Required Runtime Types

The first implementation MUST define these types.

### 6.1 IZServerRuntime

```csharp
public interface IZServerRuntime
{
    void Run(ZServerRuntimeOptions options, CancellationToken cancellationToken);
}
```

### 6.2 ZServerRuntimeOptions

```csharp
public sealed class ZServerRuntimeOptions
{
    public required string ArtifactRootPath { get; init; }
    public required TimeSpan TickInterval { get; init; }
    public required IExecutionContextFactory ExecutionContextFactory { get; init; }
    public required IConnectionSource ConnectionSource { get; init; }
}
```

---

## 7. Required Processing Model

The first implementation MUST process one connection at a time.
Parallel multi-connection processing is out of scope for the first implementation.

The implementation MUST still preserve the personal-instance semantics when parallel processing is added later.

---

## 8. Required Integration Rule

The ZBack runtime host MUST use the ZServer model as its default execution model.

This means:
- connection module creates the connection abstraction;
- session module creates per-connection session state;
- execution context module runs one personal instance;
- no multiplayer behavior exists unless multiplayer module is loaded.

---

## 9. Required Multiplayer Extension Rule

A multiplayer-capable project MUST add a built-in or explicit module that introduces:

- shared state registry;
- cross-context messaging or synchronization;
- context coordination rules.

The multiplayer module MUST be treated as additive behavior on top of personal-instance ZServer semantics.

---

## 10. Required Implementation Guidance

An implementation engineer must write the runtime as if each connection launches a separate game instance on the server side.

This is the primary mental model and must be preserved across all runtime code, module code, and docs.
