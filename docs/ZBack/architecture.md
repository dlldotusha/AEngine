# ZBack Architecture

## Purpose

ZBack is a Project Profile for Minecraft-oriented server-first development.

## Responsibilities

- define the project model
- define targets
- define project modes
- orchestrate compiler execution
- provide built-in profile modules

## Layer relationship

- ACore hosts ZBack
- AEngine provides reusable engine modules to ZBack
- ZCompiler is part of ZBack and only compiles

## Principle

ZBack does not attempt to clone the Mojang server. Mojang is only a source of constraints and interoperability limits.
