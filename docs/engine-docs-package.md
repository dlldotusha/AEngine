# AEngine Engine Docs Package

## 1. Purpose of this package

This directory is the normative technical specification for the first production implementation of AEngine.

It is written as an implementation package, not as a concept note.
The goal is that a competent engineer, Codex, or another implementation agent can build the system from these documents without relying on hidden conversation context.

This package intentionally mirrors the discipline recommended for industrial-strength engine development: explicit subsystem boundaries, layered architecture, clear runtime/tool separation, and first-class tooling and diagnostics. Those are recurring themes in Jason Gregory's discussion of engine architecture, runtime layering, core systems, and the tool plus asset pipeline split. fileciteturn3file0

---

## 2. Package structure

### Foundation
- `vision-and-scope.md`
- `terms-and-definitions.md`
- `product-and-edition-model.md`
- `repository-and-docs-layout.md`

### ACore
- `acore-spec.md`
- `acore-subsystems.md`
- `acore-plugin-host.md`
- `acore-ui-shell.md`
- `acore-diagnostics.md`
- `acore-startup-and-lifecycle.md`

### Project Profiles
- `project-profiles/README.md`
- `project-profiles/profile-system.md`
- `project-profiles/profile-manifest.md`
- `project-profiles/profile-lifecycle.md`
- `project-profiles/profile-compatibility.md`

### ZBack
- `profiles/zback/README.md`
- `profiles/zback/zback-spec.md`
- `profiles/zback/zback-cli.md`
- `profiles/zback/zback-build-pipeline.md`
- `profiles/zback/zback-run-pipeline.md`
- `profiles/zback/zback-module-integration.md`
- `profiles/zback/zback-aengine-integration.md`

### Compiler contracts
- `compiler-contract.md`
- `compiler-process-model.md`
- `compiler-diagnostics-contract.md`
- `compiler-artifact-contract.md`

### Modules
- `modules/README.md`
- `modules/module-system.md`
- `modules/module-manifest.md`
- `modules/module-lifecycle.md`
- `modules/module-dependencies-and-conflicts.md`
- `modules/module-hooks.md`
- `modules/built-in-vs-external-modules.md`

### Plugins
- `plugins/README.md`
- `plugins/plugin-system.md`
- `plugins/plugin-manifest.md`
- `plugins/plugin-permissions.md`
- `plugins/plugin-sandbox.md`
- `plugins/plugin-failure-model.md`

### Projects
- `project-format.md`
- `project-layout.md`
- `source-layout-and-naming.md`
- `execution-pipeline.md`

### Tooling
- `diagnostics-and-tooling.md`
- `debugging-model.md`
- `profiling-model.md`
- `language-services-and-highlighting.md`

### Implementation
- `implementation-roadmap.md`
- `minimum-viable-skeleton.md`
- `codex-implementation-rules.md`

---

## 3. Normative rule

All documents in this package are normative.
If one document conflicts with another, the more specific document wins.
If code conflicts with the docs, the docs win until intentionally revised.

---

## 4. Current status

This package is being expanded incrementally.
Any document marked draft is still normative unless explicitly stated otherwise.
