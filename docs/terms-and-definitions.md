# Terms and Definitions

This document is normative. All implementation work MUST use these meanings.

## ACore
Immutable host platform. Owns UI shell, profile loading, plugin hosting, permission enforcement, and diagnostics aggregation.

## Project Profile
Project-core definition. Declares project behavior, compiler binding, built-in modules, orchestration rules, and integration points.

## Compiler
External executable backend that transforms project sources into build artifacts. A compiler only compiles and validates.

## Toolchain
Command-line entrypoint and orchestration logic for a Project Profile. Example: ZBack.

## Module
Trusted compile-time extension of a Project Profile. Modules may change project capabilities, APIs, build behavior, and project semantics, but only within the owning profile’s extension system.

## Plugin
Sandboxed developer-environment extension running inside ACore. Plugins may extend tooling and UI but must not alter build output or project semantics.

## Manifest
Machine-readable descriptor for a system component. Used for discovery, validation, compatibility, and loading.

## Artifact
Any file or structured output emitted by the build process.

## Diagnostics
Normalized warnings, errors, and informational messages produced by ACore, toolchains, modules, compilers, or plugins.

## Compatibility
Formal declaration that two components may operate together.

## Dependency
Formal requirement that one component be available before another can initialize or execute.

## Conflict
Formal declaration that two components cannot be active together.

## Hook
Named extension point invoked by the owning system at a specific lifecycle or pipeline stage.

## Build Pipeline
Ordered sequence of steps executed to transform project sources into artifacts.

## Run Pipeline
Ordered sequence of steps executed to launch a built project artifact.

## Integration Layer
Code and contracts that bind a Project Profile into ACore UI and workflows.
