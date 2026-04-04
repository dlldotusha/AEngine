# ZBack Build Pipeline — Implementation Specification

## 1. Purpose

Defines exact algorithm for `zback build`.

This specification is mandatory and must be implemented without deviation.

---

## 2. High-Level Algorithm

Pseudocode (normative):

```
function build(projectPath):
    project = loadProject(projectPath)
    validateProjectSchema(project)

    profile = resolveProfile(project.profile)

    modules = loadModules(project.modules, profile)
    sortedModules = topoSort(modules)

    for module in sortedModules:
        module.validate(project)

    executionGraph = createExecutionGraph()

    invokeCompiler(project, executionGraph)

    artifacts = collectArtifacts(project.output.path)

    writeMetadata(artifacts)

    return success
```

---

## 3. Step-by-Step Specification

### 3.1 Load Project

- Read `project.zback.json`
- Deserialize into strongly typed object
- Validate required fields

Failure:
- abort
- exit code 2

---

### 3.2 Resolve Profile

- Query ProfileRegistry
- Load profile descriptor

Failure:
- abort
- exit code 2

---

### 3.3 Load Modules

Algorithm:

1. Read module ids from project
2. Resolve module manifests
3. Validate compatibility with profile
4. Construct module instances

Failure cases:
- missing module → abort
- incompatible module → abort

---

### 3.4 Topological Sort

- Build directed graph using dependencies
- Run Kahn's algorithm

Failure:
- cycle detected → abort with diagnostic

---

### 3.5 Module Validation Phase

For each module:

```
module.Validate(ProjectContext ctx)
```

Rules:
- must be deterministic
- must not mutate global state

---

### 3.6 Compiler Invocation

Command:

```
zcompiler compile --project <path> --out <out>
```

Rules:
- capture stdout/stderr
- enforce timeout (configurable)
- map exit code to pipeline state

Failure:
- exit code != 0 → abort build

---

### 3.7 Artifact Collection

- read output directory
- validate presence of required files

Failure:
- missing artifact → abort

---

### 3.8 Metadata Generation

Write file:

```
out/build-metadata.json
```

Must include:
- timestamp
- compiler id
- artifact list

---

## 4. Determinism Rules

Build MUST be deterministic if:
- input files unchanged
- config unchanged
- compiler version unchanged

---

## 5. Parallelism (future-safe)

Current implementation: sequential
Future: allow module validation parallelization

---

## 6. Logging Requirements

Each stage MUST log:
- start
- end
- duration

---

## 7. Prohibited Behaviors

- implicit module loading
- silent failure
- partial success states
