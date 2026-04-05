# Dependency Model

## Purpose

Defines how modules depend on each other.

## Dependency types

- required
- optional
- conflicting

## Rules

- dependencies must be explicit
- circular dependencies are not allowed
- optional integrations must not break base functionality

## Load order

Modules are loaded according to dependency graph resolution.
