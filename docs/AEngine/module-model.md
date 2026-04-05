# Module Model

## Purpose

Defines what an AEngine module is and how it behaves.

## Characteristics

- modular
- composable
- optionally replaceable
- profile-agnostic by default

## Module responsibilities

A module may:

- define reusable runtime behavior
- expose types and contracts
- depend on other modules explicitly
- participate in engine lifecycle through declared extension points

A module may not:

- redefine ACore
- silently modify unrelated modules
- bypass profile selection rules

## Compatibility

Modules are selected by a Project Profile. A module does not belong to ZBack just because ZBack uses it.
