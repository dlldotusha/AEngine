# Project Profiles Documentation

This directory defines the generic Project Profile system used by AEngine.

A **Project Profile** is the project-core definition. It tells AEngine and standalone tooling:

- what a project of this type is;
- what compiler backend is bound to it;
- what built-in modules are included by default;
- what project layout rules exist;
- what build and run pipelines exist;
- what runtime model is assumed by default.

A Project Profile is NOT the compiler itself.
A Project Profile is NOT ACore.
A Project Profile is NOT a plugin.

## Required distinction

- Generic profile system documents live under `docs/project-profiles/`
- Concrete profile documents live under `docs/profiles/<profile-id>/`

For example:
- `docs/project-profiles/` describes the platform-wide profile abstraction.
- `docs/profiles/zback/` describes the concrete ZBack profile.

## Runtime stance

A profile may define a default runtime model. For ZBack, the default runtime model is single-session by default: each incoming connection is treated like a separate game launch unless additional modules explicitly add shared-state or multiplayer coordination semantics.
