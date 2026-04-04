# ZBack Profile Documentation

ZBack is a concrete Project Profile for AEngine.

ZBack is responsible for providing:

- the `zback` toolchain entrypoint;
- binding to the ZCompiler backend;
- defining the default project format for ZBack projects;
- defining the runtime execution model for ZBack projects;
- shipping built-in modules required for Minecraft-oriented server projects.

## Core idea

A ZBack project compiles into a server-side target.
The default runtime model treats each connection as a new execution context, similar to launching a separate game instance for that connection.
By default, connections are isolated and identical.
Shared state, multiplayer coordination, or cross-context interaction only exist when provided by explicit modules.

## Built-in modules

ZBack includes built-in modules that provide Minecraft-oriented project functionality.
These modules are part of the concrete ZBack profile and are not part of ACore or the generic Project Profile system.

The first implementation must document and ship built-in ZBack modules explicitly rather than assuming them implicitly.

## Required companion documents

- `zback-spec.md`
- `zback-cli.md`
- `zback-build-pipeline.md`
- `zback-run-pipeline.md`
- `zback-module-integration.md`
- `zback-aengine-integration.md`
