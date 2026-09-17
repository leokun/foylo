# 0005: stable occurrence references and immutable planning basis

Status: Decided under the product owner's delegated V1 mandate. Complements [0003](0003-planned-and-observed.md) and [0004](0004-offline-first.md).

Foylo computes sessions on demand, but moves, offline declarations and historical corrections need stable references. We retain an OccurrenceRef independent of current times and planning versions: a recurring slot/person/original-date reference or an independent one-off addition reference. Planning subjects use immutable effective revisions, and declarations that infer a boundary retain their PlanningBasis so later planning edits cannot silently change earlier durations.

## Consequences and alternatives

Current activity/date or start-time identities would merge distinct slots or detach facts after a move. A mutable pre-generated occurrence table would obscure the distinction between planning and declared reality. Instead, ExpectedOccurrence, ObservedOccurrence and MonthlySummary are projections; dated exceptions and journal references keep moved, retained or historical sessions discoverable even when a recurring rule no longer generates them.

Corrections, annulments and conflict resolutions remain traceable journal actions. A pattern-derived boundary is part of a computed reading, never a fabricated Punch. Resolution records name the conflict they resolve; a new contradictory fact requires renewed resolution. This costs explicit provenance and conflict modeling, but avoids silent historical recalculation and arrival-order decisions.

The [frozen conceptual model](../02-domain-model.md) defines relationships and invariants. This decision selects no table layout, synchronization provider or access policy, and does not claim implementation or acceptance verification.
