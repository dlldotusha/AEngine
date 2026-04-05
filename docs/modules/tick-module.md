# Tick Module — Full Function Specification

## 1. Purpose

The Tick Module provides a deterministic tick counter for one execution context.

---

## 2. Required data

The module must store:

- current tick value (integer starting at 0)

---

## 3. Required functions

- `GetCurrentTick()`
- `Advance()`

---

## 4. Function: GetCurrentTick()

### Inputs

No inputs.

### Preconditions

Tick state must exist.

### Steps

1. read current tick value;
2. return value.

### Outputs

Returns current tick value.

### Side effects

None.

### Failure conditions

Must fail if tick state is not initialized.

### Call timing

May be called any time after initialization.

---

## 5. Function: Advance()

### Inputs

No inputs.

### Preconditions

Tick state must exist.

### Steps

1. read current tick value;
2. add exactly 1;
3. store updated value.

### Outputs

Returns nothing.

### Side effects

Updates internal tick value.

### Failure conditions

Must fail if state not initialized.

### Call timing

Must be called exactly once per execution context tick cycle.

### Caller

Must be called by ExecutionContext.Tick().

---

## 6. Output contract

The module guarantees strictly increasing tick values without gaps.
