# Object Registry Module — Full Function Specification

## 1. Purpose

The Object Registry Module assigns stable identifiers to runtime objects and allows retrieval by identifier.

---

## 2. Required data

The module must store:

- mapping from object id to object instance

---

## 3. Required functions

- `Register(object)`
- `Get(id)`
- `Unregister(id)`

---

## 4. Function: Register(object)

### Inputs

- object instance

### Preconditions

object exists.

### Steps

1. generate unique identifier;
2. store mapping id → object;
3. return id.

### Outputs

returns id.

### Side effects

adds entry to registry.

### Failure conditions

must fail if object is null.

---

## 5. Function: Get(id)

### Inputs

- object id

### Preconditions

id exists.

### Steps

1. lookup id;
2. return object or null.

### Outputs

object or null.

### Side effects

none.

---

## 6. Function: Unregister(id)

### Inputs

- object id

### Preconditions

id exists.

### Steps

1. remove mapping;
2. release reference.

### Outputs

none.

### Side effects

removes entry.

### Failure conditions

must fail if id invalid.

---

## 7. Output contract

Provides stable object identity mapping within one execution context.
