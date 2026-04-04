# Full Module Specifications — Strict Contracts

## 1. Purpose

This document defines strict module contracts with exact method behavior.

If a module exists, it MUST comply exactly.

---

# 2. Tick System (STRICT)

```csharp
public interface ITickSystem
{
    long CurrentTick { get; }
    void Advance();
}
```

### Required behavior

- `CurrentTick` starts at 0
- `Advance()` increments by exactly +1
- MUST NOT skip values
- MUST NOT depend on wall clock

---

# 3. Input System (STRICT)

```csharp
public interface IInputSystem
{
    InputState GetCurrent();
}
```

### Required behavior

- MUST return immutable snapshot
- MUST NOT block
- MUST NOT mutate returned object after return

---

# 4. Save System (STRICT)

```csharp
public interface ISaveSystem
{
    void Save(ExecutionContextId id, object state);
    object? Load(ExecutionContextId id);
}
```

### Required behavior

- Save MUST be atomic
- Load MUST be idempotent
- Load MUST return null if no state

---

# 5. Object Registry (STRICT)

```csharp
public interface IObjectRegistry
{
    Guid Create(object obj);
    object? Get(Guid id);
    void Remove(Guid id);
}
```

### Required behavior

- Create MUST return unique id
- Get MUST return same reference until removed
- Remove MUST make object unreachable

---

# 6. World System (STRICT OPTIONAL)

```csharp
public interface IWorldSystem
{
    object GetState();
}
```

### Required behavior

- MUST represent full world state snapshot

---

# 7. Simulation System (STRICT OPTIONAL)

```csharp
public interface ISimulationSystem
{
    void Step();
}
```

### Required behavior

- MUST advance simulation by one discrete step
- MUST be deterministic for same inputs

---

# 8. Module Composition Rule

Modules MUST:

- be stateless OR explicitly stateful
- not depend on global singletons
- not require other modules unless explicitly declared

---

# 9. ExecutionContext Integration Rule

Each module MUST be invoked from ExecutionContext.Tick() explicitly.

Implicit background execution is forbidden.
