# 0003: separation of planned and observed

Date: September 17, 2026. Status: Decided.

## Context and decision

The product must keep both what was planned and what was declared, with a usable history for summaries.

The planned is computed on demand from rules, calendars and exceptions. The observed is carried by an append-only check-in log. A correction adds a new fact instead of silently rewriting the previous one.

## Alternatives and consequences

The pre-filled occurrence table and the mutable occurrence as the single source of the observed are abandoned. The calendar becomes a dependency of the computation, to avoid counting activities when the facility is closed.

Occurrence identities, preservation of the historical schedule, concurrent corrections and the reading of incomplete data remain to be defined. The absence of a check-in does not prove a presence.
