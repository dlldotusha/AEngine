# Session Module — Strict Implementation Specification

## 1. Purpose

The Session Module stores and manages per-connection state.

Each execution context must have exactly one session.

---

## 2. What the programmer must implement

The programmer must implement:

- a session state object;
- a storage mapping from execution context id to session;
- functions to create, retrieve, and destroy sessions.

---

## 3. Required data model

The module must maintain a mapping:

ExecutionContextId → SessionState

---

## 4. Required function list

### Function: CreateSession(contextId)

Must:

1. create a new session object;
2. associate it with the given contextId;
3. store it in the mapping;
4. fail if a session already exists for that contextId.

---

### Function: GetRequired(contextId)

Must:

1. look up session by contextId;
2. return the session if it exists;
3. throw an error if session does not exist.

---

### Function: DestroySession(contextId)

Must:

1. remove the session from mapping;
2. release all resources owned by the session;
3. succeed even if called once per context lifecycle.

---

## 5. Required lifecycle usage

Order of calls per execution context:

1. CreateSession(contextId)
2. GetRequired(contextId) during runtime
3. DestroySession(contextId) on shutdown

---

## 6. Dependencies

No required dependencies.

---

## 7. Output contract

The module outputs session state tied strictly to one execution context.
No cross-context sharing is allowed.
