# Tick

## Purpose

Provides fixed-step execution.

## Responsibilities

- tick loop
- configurable tickrate
- tick callbacks
- tick-based scheduling

## Design rules

- tickrate must be configurable
- logic must not assume fixed 20 TPS
- systems should not depend on wall-clock time unless explicitly required
