# Compiler Contract Specification

## 1. Purpose

This document defines the strict contract between a Project Profile toolchain (for example, ZBack) and a compiler backend (for example, ZCompiler).

The compiler is responsible only for compilation.
It is not responsible for project orchestration, UI, plugin permissions, or profile lifecycle management.

---

## 2. Delivery Model

A compiler MUST be delivered as an external executable process.

Rationale:
- language independence;
- process isolation;
- crash containment;
- simpler standalone and CI usage.

Dynamic library integration is explicitly out of scope for the first implementation.

---

## 3. Responsibilities

The compiler MUST:
- accept declared input formats;
- parse source inputs;
- validate semantic correctness according to compiler rules;
- transform source into target artifacts;
- emit machine-readable diagnostics;
- return process exit codes.

The compiler MUST NOT:
- open windows or UI;
- own project lifecycle commands such as `new` or `run`;
- install plugins;
- manage permissions;
- assume ACore is present.

---

## 4. Invocation Model

Minimum required command surface:

```text
<compiler> compile <project-file> [options]
<compiler> validate <project-file> [options]
<compiler> info
```

Optional future commands:

```text
<compiler> explain <symbol>
<compiler> language-server
```

The first implementation MUST only rely on `compile`, `validate`, and `info`.

---

## 5. Standard Arguments

Supported arguments MUST include:

- `--project <path>` or positional project path;
- `--out <path>` output directory;
- `--diagnostics <path>` diagnostics output path;
- `--config <name>` optional build configuration;
- `--json` emit machine-readable output on stdout.

A compiler MAY support more arguments, but the above must remain stable.

---

## 6. Exit Codes

Standard exit codes:

- `0` — success;
- `1` — compilation failed due to project/source errors;
- `2` — invalid invocation or invalid arguments;
- `3` — internal compiler failure;
- `4` — environment failure (missing tool, missing dependency, inaccessible path).

ZBack and other toolchains MUST interpret exit codes exactly as defined above.

---

## 7. Diagnostics Format

The compiler MUST be able to write diagnostics to a JSON file.

Canonical schema:

```json
{
  "compilerId": "zcompiler",
  "compilerVersion": "1.0.0",
  "success": false,
  "diagnostics": [
    {
      "severity": "error",
      "code": "ZC0001",
      "message": "Unknown symbol 'Foo'",
      "file": "src/main.zb",
      "range": {
        "startLine": 10,
        "startColumn": 5,
        "endLine": 10,
        "endColumn": 8
      },
      "source": "parser"
    }
  ]
}
```

Severity levels:
- `error`
- `warning`
- `info`

Diagnostics MUST be deterministic for identical compiler version + source + configuration.

---

## 8. Artifact Contract

Compilation output MUST be written to an artifact directory.

The compiler MUST emit:
- final target artifacts;
- `build-metadata.json`;
- optional intermediate outputs if explicitly enabled.

`build-metadata.json` minimum schema:

```json
{
  "compilerId": "zcompiler",
  "compilerVersion": "1.0.0",
  "target": "minecraft-server-binary",
  "artifacts": [
    {
      "kind": "binary",
      "path": "out/server.bin"
    }
  ]
}
```

---

## 9. Input Contract

The compiler defines its own accepted source language(s) and source structure.

Examples:
- C#-like sources;
- custom DSL;
- JSON graphs;
- generated code.

ACore must not assume source language details.
ZBack may assume compiler-declared input rules only through compiler metadata.

---

## 10. Compiler Info Command

`info` MUST return static metadata in JSON:

```json
{
  "id": "zcompiler",
  "name": "ZCompiler",
  "version": "1.0.0",
  "inputKinds": ["zback-source"],
  "outputKinds": ["binary"],
  "features": ["compile", "validate"]
}
```

This is used by ZBack for compatibility validation.

---

## 11. Process Interaction Rules

- The compiler must not require stdin interaction for basic compile/validate.
- The compiler may write human-readable logs to stderr.
- Structured machine-readable output must go either to JSON files or stdout when `--json` is set.
- The compiler must never prompt interactively during normal toolchain execution.

---

## 12. Stability Rules

Compiler CLI is a compatibility boundary.
Breaking argument names, exit code meanings, or diagnostics schema is forbidden without major version change.

---

## 13. Testing Requirements

Every compiler implementation must have tests for:
- invalid invocation;
- successful compilation;
- failed compilation with diagnostics;
- deterministic output metadata;
- exit code correctness.
