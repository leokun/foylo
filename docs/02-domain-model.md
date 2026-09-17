# Domain model

## Decided

The planned side is computed from **ScheduleRule + Calendar + ScheduleException**. Expected occurrences are computed on demand, with no pre-filled occurrence table and no periodic generation job.

The observed side is described by **append-only Punches**: check-ins are added to the history, with no silent rewriting. A correction adds a fact that logically supersedes a previous one. The observed state is derived from the log.

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
- Define the identity of an occurrence. The rule/local date pair is proposed; multiple slots in a single day and one-off activities must be covered.
- Decide how a change to a rule or calendar preserves the reading of the past: validity periods, versions or another explicit strategy.
- Define the priorities between holidays, public holidays, closures and individual exceptions.
- Choose the source of French school calendars, the attachment to a zone, the update frequency and the handling of bridge days specific to an institution.
- Settle the handling of automatic starts, presumed meals and incomplete data. A planned time is not automatically an observed time.
- Define concurrent corrections, semantic duplicates and inconsistent start/end sequences.
- Distinguish the entry time on the device from the server receipt time, especially offline.

For time, the proposal is to store facts as instants, rules in local times with a reference time zone, and monthly groupings on local dates. Crossing midnight and daylight saving changes remain cases to be defined.

## Rejected/deferred

- Mutable occurrence as the single source of the observed side: replaced by the check-in log.
- Materializing several months of expected occurrences: replaced by on-demand computation.
- Full RRULE: a weekly recurrence with a validity period is proposed to start with.
- PickupRun/PickupStop in V1: an ordered list of responsibilities can be derived before introducing a round entity.
