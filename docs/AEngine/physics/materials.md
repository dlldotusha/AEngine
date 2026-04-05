# Materials

## Purpose

Defines shared material properties used across multiple engine systems.

## Responsibilities

- physical properties (density, friction, hardness)
- lighting properties (emissive, absorption)
- interaction properties (breakable, reactive)
- audio surface properties

## Design rules

- must be system-agnostic
- must integrate with physics, lighting, and interaction
- must not depend on rendering implementation
