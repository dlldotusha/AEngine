# AEngine Documentation

This directory contains the technical specification for the first implementation target of AEngine.

## Documents

- `vision-and-scope.md` — product boundary, goals, non-goals, and terminology.
- `acore-spec.md` — strict technical specification of ACore.
- `zback-spec.md` — strict technical specification of ZBack.
- `compiler-contract.md` — process contract between ZBack and a compiler backend.
- `module-system.md` — built-in module model, manifests, dependencies, conflicts, and lifecycle.
- `plugin-system.md` — plugin model, permissions, sandbox boundaries, and integration points.
- `project-format.md` — project file and workspace format.
- `diagnostics-and-tooling.md` — diagnostics, debug, profiling, logs, and IDE integration requirements.
- `implementation-roadmap.md` — staged implementation plan for Codex.

## Design stance

This documentation is intentionally written as implementation-first technical specification.
It is meant to be detailed enough that a developer with no prior context about the conversation can still implement the system correctly.

## Architectural basis

The architecture follows layered-engine principles emphasized by Jason Gregory in *Game Engine Architecture*, especially:

- separation between runtime systems and tools;
- layered architecture with downward-only dependencies;
- strong core/foundation systems;
- explicit resource and pipeline management;
- diagnostics and tooling as first-class subsystems, not afterthoughts. fileciteturn3file0
