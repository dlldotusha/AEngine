# Save Module — Full Function Specification

## 1. Purpose

The Save Module persists execution-context state across sessions.

---

## 2. Required data

The module must store:

- mapping from ExecutionContextId to serialized payload

---

## 3. Required functions

- `Save(contextId, payload)`
- `Load(contextId)`

---

## 4. Function: Save(contextId, payload)

### Inputs

- execution context id
- serialized payload

### Preconditions

1. contextId exists;
2. payload exists.

### Steps

1. validate inputs;
2. store payload in mapping;
3. replace existing payload if present.

### Outputs

None.

### Side effects

Updates persistent storage.

### Failure conditions

Must fail if payload cannot be stored.

### Call timing

Called by execution context when save is required.

---

## 5. Function: Load(contextId)

### Inputs

- execution context id

### Preconditions

contextId exists.

### Steps

1. lookup payload;
2. if exists return it;
3. if not return null.

### Outputs

payload or null.

### Side effects

None.

### Failure conditions

Must fail if storage access fails.

---

## 6. Output contract

The module guarantees correct storage and retrieval of serialized state per execution context.
