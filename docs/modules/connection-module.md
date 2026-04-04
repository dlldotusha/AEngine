# Connection Module — Strict Implementation Specification

## 1. Purpose

The Connection Module accepts incoming runtime connections and exposes them to the ZServer runtime loop.

This module is a built-in ZBack module.

---

## 2. What the programmer must implement

The programmer must implement:

- a connection abstraction type;
- a connection source type;
- one blocking method that waits for the next connection.

---

## 3. Required types

### 3.1 IRuntimeConnection

Represents one active runtime connection.

It must expose:

- a unique identifier for the connection;
- a boolean telling whether the connection is still active;
- a disposal mechanism that releases connection resources.

### 3.2 IConnectionSource

Represents the system that waits for and returns connections.

It must expose exactly one blocking method that waits for the next connection.

---

## 4. Required function list

### Function: WaitForNextConnection(cancellationToken)

This function must:

1. block until one connection is available or cancellation is requested;
2. create or retrieve the runtime connection abstraction for the accepted connection;
3. return that connection abstraction;
4. return only one connection per call.

This function must not:

- create an execution context;
- initialize gameplay state;
- call project modules other than its own internal helpers.

---

## 5. Required runtime behavior

When the runtime host asks for a connection, the module must give it one connection object.
The runtime host will then create a new execution context for that connection.

This means the module owns connection acceptance only.
It does not own game instance creation.

---

## 6. Required connection object behavior

The connection object must:

- report whether it is still active;
- remain valid until disposed;
- release all owned resources when disposed.

---

## 7. Order of use

The runtime must use this module first in the main runtime loop.

Required order:

1. wait for next connection;
2. create execution context;
3. initialize execution context;
4. tick execution context while connection active;
5. shut down execution context;
6. dispose connection.

---

## 8. Dependencies

This module has no required gameplay module dependencies.
It may depend on low-level transport code supplied by the runtime environment.

---

## 9. Output contract

The output of this module is one runtime connection object.
Nothing else is allowed as an output.
