# ZBack Runtime Host — Full Function Specification

## 1. Purpose

The ZBack Runtime Host owns the outer runtime loop of ZServer.

It accepts connections one at a time in the first implementation, creates one execution context per connection, runs that context until the connection closes, and then performs cleanup.

This document defines every required runtime-host function in a function-by-function format.

---

## 2. What the programmer must implement

The programmer must implement all of the following:

1. one runtime host type;
2. one runtime host options type;
3. one function that runs the outer accept loop;
4. one function that runs exactly one accepted connection;
5. one options validation function.

---

## 3. Required data owned by the runtime host

The runtime host implementation must use these values:

- artifact root path;
- tick interval;
- execution context factory;
- connection source;
- cancellation token supplied by caller.

---

## 4. Required public functions

The implementation must expose these functions:

- `Run(options, cancellationToken)`
- `RunSingleConnection(connection, options)`
- `ValidateOptions(options)`

---

## 5. Function: ValidateOptions(options)

### Inputs

The function receives one runtime-host options object.

### Preconditions

The function must ensure the options object exists.

### Steps

The function must perform these steps in this exact order:

1. verify the options object is not null;
2. verify artifact root path exists and is not empty;
3. verify tick interval is greater than zero;
4. verify execution context factory exists;
5. verify connection source exists;
6. complete without modifying runtime state.

### Outputs

The function does not return gameplay data.
Its output is confirmation that the runtime host options are valid.

### Side effects

This function must not mutate runtime state.

### Failure conditions

The function must fail if any required value is missing or invalid.

### Call timing

The runtime host must call this function at the beginning of `Run(options, cancellationToken)`.

### Caller

The direct caller must be the runtime host itself.

---

## 6. Function: Run(options, cancellationToken)

### Inputs

The function receives:

- one runtime-host options object;
- one cancellation token.

### Preconditions

Before doing any work, the function must ensure:

1. options object exists;
2. cancellation token is usable by caller contract;
3. options have been validated by calling `ValidateOptions(options)`.

### Steps

The function must perform these steps in this exact order:

1. call `ValidateOptions(options)`;
2. enter the outer runtime loop;
3. while cancellation has not been requested, call the connection source to wait for the next connection;
4. receive exactly one connection object;
5. call `RunSingleConnection(connection, options)`;
6. return only when cancellation is requested or an unrecoverable runtime-host failure occurs.

### Outputs

The function does not return gameplay data.
Its output is one complete runtime-host lifetime for the current ZServer process.

### Side effects

The function accepts connections and repeatedly creates and destroys execution contexts.

### Failure conditions

If waiting for the next connection fails, the function must surface the failure to runtime diagnostics and continue if the failure is recoverable.
If the failure is unrecoverable, the function must stop the outer loop.

### Call timing

This function is the top-level entrypoint for runtime execution.

### Caller

The direct caller must be the ZBack run command or equivalent runtime launcher.

---

## 7. Function: RunSingleConnection(connection, options)

### Inputs

The function receives:

- one active runtime connection object;
- one validated runtime-host options object.

### Preconditions

Before doing any work, the function must ensure:

1. connection exists;
2. options exist;
3. execution context factory exists;
4. tick interval is valid.

### Steps

The function must perform these steps in this exact order:

1. create one new execution context by calling the execution context factory;
2. initialize that execution context;
3. while the connection remains active, call the execution context tick function exactly once per loop iteration;
4. after each tick, wait exactly one tick interval;
5. when the connection becomes inactive, call execution context shutdown;
6. dispose the connection object;
7. return control to the outer runtime loop.

### Outputs

The function does not return gameplay data.
Its output is one fully completed personal-instance lifecycle for one connection.

### Side effects

The function creates one execution context and causes one complete personal-instance runtime to run.

### Failure conditions

If execution context creation fails:

1. dispose the connection;
2. return control to the outer loop.

If execution context initialization fails:

1. attempt execution context shutdown if a context object was created;
2. dispose the connection;
3. return control to the outer loop.

If one tick fails:

1. stop ticking immediately;
2. call execution context shutdown;
3. dispose the connection;
4. return control to the outer loop.

If shutdown fails:

1. still dispose the connection;
2. return control to the outer loop.

### Call timing

This function is called once for every accepted connection.

### Caller

The direct caller must be `Run(options, cancellationToken)`.

---

## 8. Required order of operations

The runtime host must follow this order exactly:

1. validate options;
2. wait for one connection;
3. create one execution context;
4. initialize one execution context;
5. tick one execution context repeatedly;
6. shut down one execution context;
7. dispose one connection;
8. return to waiting for the next connection.

---

## 9. Output contract

The runtime host outputs exactly one isolated personal runtime instance per connection.
It must not create cross-context shared state by default.
