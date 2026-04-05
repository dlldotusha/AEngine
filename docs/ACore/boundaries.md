# ACore Boundaries

## Owns

- host lifecycle
- editor shell foundation
- profile loading contracts
- module loading contracts
- plugin loading contracts
- plugin permission enforcement
- shared diagnostics base

## Does not own

- engine subsystem semantics
- profile-specific rules
- compiler implementation details
- gameplay runtime design
- build orchestration for specific project profiles

## Boundary rule

If a concern is reusable engine behavior, it belongs to AEngine.
If a concern is profile-specific project behavior, it belongs to a Project Profile such as ZBack.
If a concern only improves development workflow, it belongs to plugins.
