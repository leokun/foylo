# Foylo

Foylo coordinates everyday family responsibilities and distinguishes plans, declarations and inferences about attendance. The [domain model](docs/02-domain-model.md) holds the broader conceptual model still under discussion.

## Language

**Punch**:
A declared fact about attendance or timing, including a correction to an earlier declaration. A value inferred from the plan is not a Punch.
_Avoid_: Inferred check-in, automatic declaration

**Presumed meal**:
A planned school lunch treated as taken once its planned slot has ended, without an explicit check-in or a contrary declaration. Its presumed status distinguishes it from explicitly declared attendance.
_Avoid_: Confirmed meal, declared presence

**Estimated start**:
The planned start used to calculate after-school-care duration when only the pick-up time has been declared. It is an estimate, not a declared arrival.
_Avoid_: Actual arrival, automatic check-in

**Estimated duration**:
A care duration calculated from one declared boundary and one boundary supplied by the pattern: the start for afternoon care or the end for morning care. Its provenance distinguishes it from a duration whose start and end were both declared.
_Avoid_: Fully declared duration

**Declared absence**:
An explicit declaration that a person did not attend an activity. For school lunch, it replaces a presumed meal.
_Avoid_: Missing check-in, planned cancellation

**Needs completion**:
An explicitly declared attendance record whose duration cannot be calculated because a required time is missing or its timing is inconsistent. A valid estimated start does not by itself make the record incomplete; neither does a morning-care slot with no entry declaration or an afternoon-care slot with no exit declaration.
_Avoid_: Zero duration, absence

**Incomplete total**:
A duration total that excludes one or more records marked Needs completion. It also excludes durations affected by unresolved suspected duplicates and must be distinguished from a complete total for the same period.
_Avoid_: Final total, complete duration

**Annulment**:
A recorded withdrawal of a check-in entered by mistake, with its history retained. It is distinct from declaring absence or cancelling a planned activity.
_Avoid_: Deletion, absence, schedule cancellation

**Suspected duplicate**:
Two independent declarations of the same type for the same child and session, flagged for attention without automatic selection of which to retain. Repeated transmission of one operation is a separate technical case.
_Avoid_: Confirmed duplicate, automatic merge

**Declared duration**:
A duration calculated from a declared arrival and a declared pick-up. It is distinguished from a duration using an estimated start.
_Avoid_: Estimated duration, verified duration

**Effective date**:
The date from which a weekly-template change applies. Earlier sessions and their calculations retain their prior planning basis unless explicitly corrected.
_Avoid_: Silent retroactive change

**Historical correction**:
An explicit, traceable change to past information. It is distinct from applying an ordinary weekly-template change to future sessions.
_Avoid_: Automatic historical recalculation

**Occurrence**:
A particular session of an activity for a person, distinct from other slots on the same day and from a one-off addition. Its identity and attached declarations and exceptions persist when its time changes without changing its day.
_Avoid_: Activity/date pair, start-time identity

**Care exit**:
The parent's declaration that a child is leaving after-school care at a given time. It supplies the end of an interval whose start is automatically taken from the applicable pattern, with both origins distinguishable.
_Avoid_: Mandatory arrival check-in, generic pick-up requirement

**Normal school pick-up**:
The assumed outcome of the school/care pattern when no care exit is declared. It does not constitute a verified collection or an automatically authored check-in.
_Avoid_: Verified pick-up, presumed care attendance

**Care entry**:
The parent's declaration that a child enters morning care at a given time. It supplies the start of an interval whose end is automatically taken from the school-start time in the applicable pattern.
_Avoid_: Mandatory morning care exit

**Estimated end**:
The school-start time used as the end of a morning-care interval following a declared Care entry. It comes from the pattern rather than an independently declared exit.
_Avoid_: Observed exit, declared exit

**Normal school arrival**:
The assumed outcome of the morning school/care pattern when no Care entry is declared. It is not a verified arrival or an automatically authored check-in.
_Avoid_: Verified arrival, presumed morning care attendance
