# Domain model

## Decided

The planned side is computed from **ScheduleRule + Calendar + ScheduleException**. Expected occurrences are computed on demand, with no pre-filled occurrence table and no periodic generation job.

The observed side is described by **append-only Punches**: check-ins are added to the history, with no silent rewriting. A correction adds a fact that logically supersedes a previous one. The observed state is derived from the log.

A school lunch with no check-in becomes **presumed taken** at the end of its planned slot, unless explicitly declared otherwise. Before then it remains planned and is excluded from the taken-meal total. Presumed attendance is distinct from explicitly declared attendance. An end-only after-school-care check-in uses the planned start to calculate duration, with that start explicitly marked **estimated**. Neither inference is a declared Punch. The resulting duration retains its estimated-start provenance. See the [confirmed rules](01-v1-scope.md#confirmed-plannedobserved-rules) and the [domain glossary](../CONTEXT.md).

An explicitly declared lunch absence replaces a presumed meal. A subsequently declared arrival replaces the estimated start in the calculation, preserving history. These rules do not yet arbitrate conflicting explicit declarations.

For an explicitly declared care event, missing required times or inconsistent timing produce a **Needs completion** result. Its duration is excluded from the calculated total, which is identified as incomplete. A valid estimated start remains usable under the confirmed end-only rule; lack of an actual arrival declaration alone does not make that duration incomplete. For morning care, a usable pattern-derived end similarly removes the need for a separate exit declaration.

A mistaken Punch may be annulled while its history remains available. The append-only history must retain that annulment rather than silently removing the original declaration. After annulment, the state is recalculated from the remaining information. For afternoon care with a remaining exit declaration, annulling a separately declared arrival restores the pattern-derived start. Annulling the only morning entry or afternoon exit restores the normal school arrival or pick-up assumption. Annulment of a lunch absence restores a presumed meal after the planned slot ends if the planned lunch still applies and no other declaration overrides it; before then it restores the planned state. The representation of annulment remains to be defined.

Two declarations of the same type for the same child and session are a **Suspected duplicate**. They are flagged without automatically selecting a declaration to retain. Resolution explicitly selects the correct declaration and annuls the other while retaining history. Until resolution, the affected duration is excluded from the total and the summary is marked incomplete. This does not conflate a repeated transmission of one operation with two independent declarations.

The monthly summary distinguishes declared and presumed meals, and fully declared and estimated durations, with a count of sessions that need completion. The duration of an unresolved suspected duplicate is excluded, with an incomplete-summary indicator.

A change to the weekly template takes effect from a chosen date. Sessions before that effective date and their calculations retain the planning information that applied to them. Correcting the past is a separate, explicit and traceable action. This principle was confirmed in [Occurrence identity and preservation of the past when rules or calendars change](https://github.com/leokun/foylo/issues/8); the representation of versions or validity periods is not yet selected.

School-calendar updates also preserve past sessions and their calculations. Applying a retroactive calendar correction must be explicit and traceable.

Occurrence identity distinguishes separate slots on the same day, even for the same activity. A one-off activity has its own occurrence identity. A time change within the same day preserves the session identity and its attached Punches and ScheduleExceptions. Preserving attachment does not override the protection of historical calculations or silently change declared times. The exact identity representation and cross-day moves remain to be decided.

A school/care pattern defines school dismissal and a possible care start. With school ending at 16:30 and care starting at 16:45, no action means normal school pick-up is assumed, not automatic care attendance. Selecting **Care exit** at 17:00 automatically records a 16:45-17:00 interval, distinguishing the pattern-derived start from the declared end. A scheduled slot without an exit is not an incomplete care record. The event representation remains open; a generic child-pick-up event is not required by this interaction.

Morning care uses the complementary interaction: a declared Care entry at 07:45 and an 08:30 school start in the pattern automatically record both interval boundaries and 45 minutes of care. The start is declared and the end is pattern-derived. No entry declaration means normal school arrival is assumed, without recorded care or an incomplete-care warning. The morning pattern-derived end, like the afternoon pattern-derived start, must remain distinct from an independently declared fact.

## To validate: conceptual entities

| Concept | Responsibility |
| --- | --- |
| Family | Household and sharing scope |
| User | Authenticated account |
| FamilyMembership | Membership of an account in the household, role and status |
| Person | Represented person, with or without an account |
| Place | Registered place, address and optional coordinates |
| Activity | School lunch, after-school care, daycare, nanny or other activity |
| Calendar | Opening periods, holidays, public holidays and closures |
| ScheduleRule | Weekdays, validity period, local times and calendar |
| ScheduleException | One-off change to the planned side, with an optional reason |
| Assignment | Responsibility for drop-off, pick-up or care |
| Punch | Declared fact, author, device and optional correction |
| ExpectedOccurrence | Computed result of the planned side for a date |
| ObservedOccurrence | Reading of the facts applicable to that occurrence |

A Person must be able to exist without a User and later receive an account without losing their history.

For Punch, the fields under consideration are the identifier, the household, the person, the activity, the type START/END/PRESENT/ABSENT, the declared time effectiveAt, the entry time recordedAt, the author recordedBy, the device deviceId and a supersedesId reference for corrections. This is not yet a definitive schema.

## To validate: structuring rules

- Distinguish the anticipated cancellation of the planned side, carried by an exception, from an absence declared after the fact, carried by a check-in.
- Select an occurrence-identity representation that distinguishes daily slots and one-off additions and stays stable across same-day time changes. A plain activity/date or start-time-based identity cannot satisfy these requirements. Define cross-day moves separately.
- Choose a representation for planning history, such as dated versions or validity periods. Weekly-template changes and calendar updates must preserve the past; same-day time changes retain attached exceptions and declarations. Define cross-day moves and the treatment of sessions removed by a future plan change.
- Define the priorities between holidays, public holidays, closures and individual exceptions.
- Choose the source of French school calendars, the attachment to a zone, the update frequency and the handling of bridge days specific to an institution.
- Define the treatment of contradictory explicit attendance declarations and competing resolutions. The monthly provenance breakdown, completion count and exclusion of unresolved duplicate durations are confirmed.
- Define concurrent corrections and competing duplicate resolutions. Normal duplicate resolution is explicit selection plus annulment; annulment recalculates the remaining state without deleting history.
- Distinguish the entry time on the device from the server receipt time, especially offline.

For time, the proposal is to store facts as instants, rules in local times with a reference time zone, and monthly groupings on local dates. Crossing midnight and daylight saving changes remain cases to be defined.

## Rejected/deferred

- Mutable occurrence as the single source of the observed side: replaced by the check-in log.
- Materializing several months of expected occurrences: replaced by on-demand computation.
- Full RRULE: a weekly recurrence with a validity period is proposed to start with.
- PickupRun/PickupStop in V1: an ordered list of responsibilities can be derived before introducing a round entity.
