# Recommended Modules — Code-Level Specification

## 1. Purpose

Defines recommended modules grouped by importance level.

Each module is defined with exact responsibilities and required method behavior.

---

## 2. Categories

Modules are divided into:

- Core (must exist in almost every project)
- Recommended (common but optional)
- Optional (advanced systems)

---

# CORE MODULES

## 3. Tick System Module

### 3.1 Responsibility

Provide a stable tick abstraction for execution context.

### 3.2 Required Interface

```csharp
public interface ITickSystem
{
    long CurrentTick { get; }
    void Advance();
}
```

### 3.3 Required Behavior

- `Advance()` increments tick by 1
- must be called exactly once per RuntimeHost loop iteration

---

## 4. Input Module

### 4.1 Responsibility

Provide per-connection input abstraction.

### 4.2 Required Interface

```csharp
public interface IInputSystem
{
    InputState GetCurrent();
}

public sealed class InputState
{
    public required IReadOnlyDictionary<string, object> Values { get; init; }
}
```

### 4.3 Required Behavior

- returns latest input snapshot
- must not block

---

# RECOMMENDED MODULES

## 5. Save System Module

### 5.1 Responsibility

Persist execution context state.

### 5.2 Required Interface

```csharp
public interface ISaveSystem
{
    void Save(ExecutionContextId id, object state);
    object? Load(ExecutionContextId id);
}
```

### 5.3 Required Behavior

- Save must overwrite previous state
- Load must return null if no data exists

---

## 6. Object System Module

### 6.1 Responsibility

Provide runtime object registry.

### 6.2 Required Interface

```csharp
public interface IObjectRegistry
{
    Guid Create(object obj);
    object? Get(Guid id);
    void Remove(Guid id);
}
```

---

# OPTIONAL MODULES

## 7. World Module

### 7.1 Responsibility

Provide world abstraction.

### 7.2 Required Interface

```csharp
public interface IWorldSystem
{
    object GetState();
}
```

---

## 8. Simulation Module

### 8.1 Responsibility

Run advanced simulations.

### 8.2 Required Interface

```csharp
public interface ISimulationSystem
{
    void Step();
}
```

---

## 9. Rule

Modules MUST be independent.
Modules MUST not assume presence of other modules unless declared.
