# Input Module — Full Function Specification

## 1. Purpose

The Input Module captures the current input state of one active connection for one execution context.

The module must provide one immutable input snapshot per tick.

---

## 2. What the programmer must implement

The programmer must implement:

1. one input snapshot type;
2. one capture function;
3. one validation rule for snapshot completeness.

---

## 3. Required data owned by the module

The module must use:

- one input snapshot object per capture operation;
- one input field collection inside the snapshot.

The snapshot must be immutable after creation.

---

## 4. Required public functions

The implementation must expose these functions:

- `Capture(connection)`
- `ValidateSnapshot(snapshot)`

---

## 5. Function: Capture(connection)

### Inputs

The function receives one active runtime connection.

### Preconditions

Before doing any work, the function must ensure:

1. the connection exists;
2. the connection is active.

### Steps

The function must perform these steps in this exact order:

1. read all currently available input values from the connection;
2. create one new input snapshot object;
3. copy the read input values into the snapshot object;
4. call `ValidateSnapshot(snapshot)`;
5. return the snapshot.

### Outputs

The function must return one immutable input snapshot.

### Side effects

The function must not mutate execution-context gameplay state.

### Failure conditions

The function must fail if:

- the connection is missing;
- the connection is inactive when input capture is attempted;
- the snapshot cannot be validated.

### Call timing

The execution context must call this function once per tick if the Input Module is loaded.

### Caller

The direct caller must be `ExecutionContext.Tick(context)`.

---

## 6. Function: ValidateSnapshot(snapshot)

### Inputs

The function receives one input snapshot.

### Preconditions

The function must ensure the snapshot exists.

### Steps

The function must perform these steps:

1. verify the snapshot object exists;
2. verify the snapshot field collection exists;
3. verify the snapshot can be treated as immutable for the rest of the current tick;
4. complete without altering snapshot contents.

### Outputs

The function does not return gameplay data.
Its output is confirmation that the snapshot is valid.

### Side effects

None.

### Failure conditions

The function must fail if:

- the snapshot is missing;
- the field collection is missing;
- the snapshot is not safe to treat as immutable.

### Call timing

The function must be called inside `Capture(connection)` before returning the snapshot.

---

## 7. Output contract

The output of this module is exactly one immutable input snapshot representing the current input state for one tick of one execution context.
