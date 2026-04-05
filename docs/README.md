# AEngine Docs

This folder contains the architecture documentation for AEngine.

## Top-level structure

- `ACore/` — the immutable platform foundation.
- `ZBack/` — the project profile for Minecraft-oriented server targets.
- `modules/` — module documentation.
  - `engine/` — reusable engine modules that belong to AEngine.
  - `service/` — ZBack-specific service and target modules.
- `plugins/` — development-only plugins.
  - `module-plugins/` — UI/tooling plugins attached to modules.
  - `general/` — general-purpose development plugins.

## Core architectural rule

AEngine is split into three major layers:

1. `ACore` — platform and host.
2. `AEngine` — reusable engine modules.
3. `Project Profile` — concrete project model, compiler orchestration, built-in modules, rules.

For ZBack specifically:

- `ACore` hosts the environment.
- `ZBack` defines the project profile.
- `ZCompiler` is part of ZBack and only compiles.
- plugins never participate in compilation.
