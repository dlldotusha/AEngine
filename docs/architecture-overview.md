# Architecture Overview

AEngine is structured as a layered system:

1. ACore — platform and host
2. AEngine — reusable engine modules
3. Project Profile — project definition and toolchain

## Flow

Developer → ACore → Project Profile (ZBack) → Compiler → Output (Docker / Server)

## Principles

- strict separation of concerns
- modular engine systems
- plugins affect development only
- profiles define project semantics
- compilers are isolated from UI and plugins
