# Events

## Purpose

The Events module provides the messaging and event-dispatch foundation used by higher-level engine systems.

## Responsibilities

- publish events
- subscribe to events
- queue events for later processing
- provide local system-to-system messaging

## Design rules

- event dispatch must not create hidden global ownership
- queued and immediate dispatch should be separable
- event payloads should remain stable and versionable where possible
