# 0004: offline operation

Date: September 17, 2026. Status: Decided.

## Context and decision

An adult must be able to declare a pick-up in front of the school even with a poor connection. Local recording precedes synchronization and the interface immediately reflects the entry.

## Alternatives and consequences

An online-only validation does not meet the need. Synchronization will have to handle retries, repeated sends, duplicate declarations and rejected operations.

The append-only log provides the necessary history without deciding on its own the business resolution of conflicts. Local storage, the synchronization engine and the resolution rules remain to be chosen.

## Subsequent clarification

The [V1 business decisions](../12-v1-business-decisions.md) select explicit resolution of conflicting facts, and [0005](0005-occurrence-references-and-planning-basis.md) records the required identity and provenance model. Local storage and synchronization technology remain open.
