# Execution Context Module — Full Function Specification

## 1. Purpose

The Execution Context Module owns one complete personal runtime instance for one active connection.

One active connection MUST map to exactly one execution context.
One execution context MUST map to exactly one session.
One execution context MUST own exactly one ordered module invocation plan.

This module is the runtime composition root for project-level modules inside a single ZServer connection lifecycle.

---

## 2. What the programmer must implement

The programmer must implement all of the following:

1. one execution context state object;
2. one initialization function;
3. one single-step tick function;
4. one shutdown function;
5. one fixed module order resolver;
6. one per-context service/module access object.

---

## 3. Required data owned by the module

The execution context implementation must own these values:

- `ExecutionContextId ContextId`
- connection reference
- session reference
- module registry or service registry reference
- per-context mutable state object
- current lifecycle state
- ordered list of active module handles

---

## 4. Required public functions

The implementation must expose these public functions:

- `Initialize(context)`
- `Tick(context)`
- `Shutdown(context)`
- `ResolveModuleOrder(context)`

---

## 5. Function: Initialize(context)

### Inputs

The function must receive one context input object containing at minimum:

- execution context id;
- active connection reference;
- session reference;
- module/service registry.

### Preconditions

Before doing any work, the function must ensure:

1. the context object exists;
2. the context id exists;
3. the connection exists;
4. the session exists;
5. the registry exists;
6. the execution context lifecycle state is `Created`.

### Steps

The function must perform these steps in this exact order:

1. store the context id on the execution context instance;
2. store the connection reference;
3. store the session reference;
4. store the registry reference;
5. create the per-context state container;
6. resolve the ordered module invocation plan by calling `ResolveModuleOrder(context)`;
7. store the resolved module order;
8. mark the lifecycle state as `Initialized`.

### Outputs

The function does not return project data.
The output of the function is a fully prepared execution context ready for the first call to `Tick(context)`.

### Side effects

The function creates per-context runtime state and binds runtime dependencies to this execution context instance.

### Failure conditions

The function must fail if:

- any required input is missing;
- the lifecycle state is not `Created`;
- required modules cannot be resolved;
- module order cannot be built.

If the function fails, it must leave the execution context in a non-running state.

### Call timing

The runtime host must call this function exactly once per accepted connection, before the first tick.

### Caller

The direct caller must be the runtime host or execution context factory output handler.

---

## 6. Function: ResolveModuleOrder(context)

### Inputs

The function receives the same context object used for initialization.

### Preconditions

The function must ensure:

1. registry exists;
2. built-in required modules are available;
3. any optional project modules have already been loaded into the registry.

### Steps

The function must perform these steps:

1. create an empty ordered list;
2. inspect loaded modules from the registry;
3. add modules in the canonical order below if present;
4. skip modules that are not loaded;
5. store no duplicates;
6. return the ordered list.

### Canonical order

The canonical order must be:

1. Input Module
2. Tick Module
3. Time Module
4. Event Bus Module
5. Simulation Module
6. World Module
7. Entity Module
8. AI Processing Module
9. Inventory Module
10. Save Module
11. Diagnostics Module

### Outputs

The function must return one ordered list of module handles.

### Side effects

This function must not modify gameplay state.

### Failure conditions

The function must fail if a required built-in module for the profile cannot be resolved.

### Call timing

This function must be called during `Initialize(context)` before the lifecycle state becomes `Initialized`.

---

## 7. Function: Tick(context)

### Inputs

The function receives the initialized execution context input object.

### Preconditions

Before doing any work, the function must ensure:

1. lifecycle state is `Initialized` or `Running`;
2. connection exists;
3. connection is active;
4. session exists;
5. ordered modules exist.

### Steps

The function must perform exactly one logical runtime step and then return.

It must execute the following steps in this order:

1. if lifecycle state is `Initialized`, change it to `Running`;
2. call the Input Module for the current connection if present;
3. call the Tick Module if present;
4. call the Time Module if present;
5. call the Event Bus Module if present;
6. call the Simulation Module if present;
7. call the World Module if present;
8. call the Entity Module if present;
9. call the AI Processing Module if present;
10. call the Inventory Module if present;
11. call the Save Module if present when the save policy requires it;
12. call the Diagnostics Module if present to record tick-level diagnostics;
13. return control to the caller.

### Outputs

The function does not return gameplay data.
Its output is one completed logical step of the personal game instance.

### Side effects

The function may mutate per-context state and module-owned state for this execution context.

### Failure conditions

If any module call fails:

1. the function must stop processing remaining modules for that tick;
2. the failure must be reported to runtime failure handling;
3. control must return to the runtime host so that shutdown can occur.

### Call timing

The runtime host must call this function repeatedly while the connection is active.

### Caller

The direct caller must be the runtime host loop.

---

## 8. Function: Shutdown(context)

### Inputs

The function receives the current execution context input object.

### Preconditions

The function may be called when lifecycle state is `Initialized`, `Running`, or `ShuttingDown`.

### Steps

The function must perform these steps in this order:

1. if lifecycle state is already `Terminated`, return immediately;
2. set lifecycle state to `ShuttingDown`;
3. release module-owned cleanup resources for this context if they exist;
4. clear the ordered module list;
5. release per-context state container;
6. detach session reference from the execution context object;
7. detach connection reference from the execution context object;
8. set lifecycle state to `Terminated`.

### Outputs

The function does not return gameplay data.
Its output is one fully terminated execution context.

### Side effects

The function releases all execution-context-owned resources.

### Failure conditions

The function must attempt best-effort cleanup even if one cleanup step fails.
The function must not leave the execution context in `Running` state after cleanup starts.

### Call timing

The runtime host must call this function exactly once after the connection becomes inactive or after a tick failure.

### Caller

The direct caller must be the runtime host.

---

## 9. Dependencies

This module requires these modules to exist for the first implementation:

- Connection Module
- Session Module

This module may use these modules if loaded:

- Input Module
- Tick Module
- Time Module
- Event Bus Module
- Simulation Module
- World Module
- Entity Module
- AI Processing Module
- Inventory Module
- Save Module
- Diagnostics Module

---

## 10. Output contract

The module outputs exactly one fully managed personal game-instance lifecycle for one connection.
By default, it must not expose shared state across execution contexts.
