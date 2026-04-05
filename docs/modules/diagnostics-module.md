# Diagnostics Module — Full Function Specification

## 1. Purpose

The Diagnostics Module records runtime diagnostics for one execution context.

It is responsible for turning runtime events, warnings, and failures into structured diagnostic entries.

---

## 2. What the programmer must implement

The programmer must implement:

1. one diagnostic entry type;
2. one diagnostic storage collection or sink;
3. one function for informational diagnostics;
4. one function for warning diagnostics;
5. one function for error diagnostics;
6. one function for snapshot retrieval.

---

## 3. Required data owned by the module

The module must own:

- one collection of diagnostic entries for the current execution context or runtime sink;
- one timestamp source for diagnostic creation;
- one severity marker for each entry.

---

## 4. Required public functions

The implementation must expose these functions:

- `Info(code, message)`
- `Warning(code, message)`
- `Error(code, message)`
- `Snapshot()`

---

## 5. Function: Info(code, message)

### Inputs

- diagnostic code
- diagnostic message

### Preconditions

1. code exists;
2. message exists.

### Steps

1. validate the inputs;
2. create one diagnostic entry with severity `info`;
3. attach current timestamp;
4. store the entry in the diagnostic collection or forward it to the sink.

### Outputs

No gameplay data is returned.

### Side effects

Adds one informational diagnostic entry.

### Failure conditions

Must fail if the diagnostic system is unavailable.

### Call timing

May be called during initialization, ticking, or shutdown.

---

## 6. Function: Warning(code, message)

### Inputs

- diagnostic code
- diagnostic message

### Preconditions

1. code exists;
2. message exists.

### Steps

1. validate the inputs;
2. create one diagnostic entry with severity `warning`;
3. attach current timestamp;
4. store the entry in the diagnostic collection or forward it to the sink.

### Outputs

No gameplay data is returned.

### Side effects

Adds one warning diagnostic entry.

### Failure conditions

Must fail if the diagnostic system is unavailable.

### Call timing

May be called during initialization, ticking, or shutdown.

---

## 7. Function: Error(code, message)

### Inputs

- diagnostic code
- diagnostic message

### Preconditions

1. code exists;
2. message exists.

### Steps

1. validate the inputs;
2. create one diagnostic entry with severity `error`;
3. attach current timestamp;
4. store the entry in the diagnostic collection or forward it to the sink.

### Outputs

No gameplay data is returned.

### Side effects

Adds one error diagnostic entry.

### Failure conditions

Must fail if the diagnostic system is unavailable.

### Call timing

May be called during initialization, ticking, shutdown, or runtime failure handling.

---

## 8. Function: Snapshot()

### Inputs

No inputs.

### Preconditions

The diagnostic collection or sink access must exist.

### Steps

1. read all stored diagnostic entries;
2. create one stable snapshot representation;
3. return the snapshot.

### Outputs

Returns a collection of diagnostic entries.

### Side effects

None.

### Failure conditions

Must fail if diagnostics cannot be read.

### Call timing

May be called after any diagnostics have been emitted.

---

## 9. Output contract

The module outputs structured diagnostic entries with deterministic severity, code, message, and timestamp fields.
