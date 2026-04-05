# Glossary

## ACore
Immutable platform foundation of AEngine.

## AEngine
Reusable engine module layer.

## Project Profile
A concrete project model built on top of ACore and AEngine.

## ZBack
Minecraft-oriented Project Profile.

## Compiler
Low-level compilation backend inside a Project Profile.

## Module
A compile/runtime capability unit. Engine modules belong to AEngine. Service modules belong to a profile such as ZBack.

## Plugin
Development-only extension. Plugins do not participate in compilation.

## Module Plugin
A plugin that provides UI or tooling for a specific module.

## Target
A build/deployment target defined by a profile.

## Mode
A predefined runtime/project mode such as Offline or Online Client-Server.

## Manifest
A metadata document that describes profiles, modules, plugins, or related installable units.

## Artifact
A build output file or package.

## Toolchain
The set of tools used to build and run a project.

## Orchestrator
The component that coordinates build and runtime workflow steps.

## Execution Context
The active runtime context in which project logic executes.

## Built-in Module
A module included by default in a Project Profile.
