# AEngine

AEngine is the reusable engine layer of the overall platform.

It contains engine modules that are not tied to any single Project Profile.

## Responsibilities

- provide reusable engine subsystems
- define engine-side module boundaries
- support composition of simulation, physics, gameplay, visuals, and AI systems

## Non-responsibilities

AEngine does not:

- replace ACore
- define project-specific targets
- define profile-specific service behavior
- define compiler orchestration for ZBack or any other profile

## Architectural rule

If a system is reusable across multiple project profiles, it belongs in AEngine.
If it is specific to a profile such as ZBack, it does not belong in AEngine.
