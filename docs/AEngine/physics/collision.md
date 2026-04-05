# Collision

## Purpose

Provides collision detection as an independent engine system.

## Responsibilities

- shape overlap checks
- ray and query tests
- contact generation
- collision boundaries

## Design rules

- collision is independent from rigid body simulation
- collision must be reusable by gameplay, AI, and movement
- collision should not assume a single world implementation
