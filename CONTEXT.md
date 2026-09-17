# Foylo

Foylo coordinates everyday family responsibilities and distinguishes plans, declarations and inferences about attendance. The [domain model](docs/02-domain-model.md) holds the frozen V1 conceptual baseline and its relationship invariants.

## Language

**Punch**:
An attendance journal record: a declaration, correction, annulment or explicit resolution of competing declarations. A value inferred from the plan is not a Punch.
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
A quantity or duration total whose reading excludes affected sessions because of missing information, conflicting declarations or unresolved planning. It is distinct from the synchronization status of the underlying records.
_Avoid_: Final total, complete duration

**Annulment**:
A recorded withdrawal of a check-in entered by mistake, with its history retained. It is distinct from declaring absence or cancelling a planned activity.
_Avoid_: Deletion, absence, schedule cancellation

**Suspected duplicate**:
Two independent declarations of the same type for the same child and session, flagged for attention without automatic selection of which to retain. Repeated transmission of one operation is a separate technical case.
_Avoid_: Confirmed duplicate, automatic merge

**Declared duration**:
A duration calculated from a declared arrival and a declared pick-up. It is distinguished from a duration using a pattern-derived start or end.
_Avoid_: Estimated duration, verified duration

**Effective date**:
The date from which a weekly-template change applies. Earlier sessions and their calculations retain their prior planning basis unless explicitly corrected.
_Avoid_: Silent retroactive change

**Historical correction**:
An explicit, traceable change to past information. It is distinct from applying an ordinary weekly-template change to future sessions.
_Avoid_: Automatic historical recalculation

**Occurrence**:
A particular session of an activity for a person, distinct from other slots on the same day and from a one-off addition. Its identity persists across permitted time and date changes; a recurring session keeps its original slot, person and date reference while a one-off session has an independent identity.
_Avoid_: Activity/date pair, start-time identity

**Care exit**:
The parent's declaration that a child leaves care at a given time. For afternoon care it combines with the pattern-derived start; for morning care it may replace the pattern-derived end of an interval with a declared entry.
_Avoid_: Mandatory arrival check-in, generic pick-up requirement

**Normal school pick-up**:
The assumed outcome of the school/care pattern when no care exit is declared. It does not constitute a verified collection or an automatically authored check-in.
_Avoid_: Verified pick-up, presumed care attendance

**Care entry**:
The parent's declaration that a child enters morning care at a given time. It supplies the start of an interval whose end comes from the school-start time in the applicable pattern unless an explicit morning Care exit replaces it.
_Avoid_: Mandatory morning care exit

**Estimated end**:
The school-start time used as the end of a morning-care interval following a declared Care entry. It comes from the pattern rather than an independently declared exit.
_Avoid_: Observed exit, declared exit

**Normal school arrival**:
The assumed outcome of the morning school/care pattern when no Care entry is declared. It is not a verified arrival or an automatically authored check-in.
_Avoid_: Verified arrival, presumed morning care attendance

**To resolve**:
The planning status of a session with incompatible or unresolved planning information, including an exception whose recurring slot was removed. It remains visible until an explicit planning decision resolves the incompatibility.
_Avoid_: Automatic cancellation, Needs completion

**Family**:
The household that owns a shared set of people, activities, planning and attendance history. It defines the membership boundary and reference time zone.
_Avoid_: User account

**User**:
An authenticated account that may represent a Person and gain household access through a FamilyMembership.
_Avoid_: Person, household

**FamilyMembership**:
The relationship that grants a User a role and access status within a Family.
_Avoid_: Responsible adult assignment

**Person**:
A child or adult represented in a Family, whether or not they have an account. Their identity remains the same when they later receive an account.
_Avoid_: User

**Place**:
A named location used for activities or sessions within a Family.
_Avoid_: Live location

**Activity**:
A household activity definition whose interpretation determines whether attendance is counted as a quantity or an interval.
_Avoid_: Occurrence, calendar

**Calendar**:
The opening and closure policy applicable to an activity and place, including known coverage and the provenance of its information.
_Avoid_: Weekly template, proof of attendance

**ScheduleRule**:
A stable recurring slot for one Person and Activity, whose revisions define its schedule and planning context.
_Avoid_: Generated session, whole household template

**ScheduleException**:
A dated addition, cancellation, move or override of planned participation. It does not declare what actually happened.
_Avoid_: Declared absence, correction of attendance

**Assignment**:
The responsibility of a named adult Person for a task such as drop-off or pick-up, either recurring or specific to one occurrence.
_Avoid_: Access permission, Punch author

**PlanningRevision**:
An immutable version of a planning decision, with its effective dates and traceable origin.
_Avoid_: Silent overwrite

**OccurrenceRef**:
The stable reference to an Occurrence, retained across its permitted changes of time or date.
_Avoid_: Current date/time identity

**PlanningBasis**:
The set of planning revisions and dependencies used to explain a computed session or inferred boundary.
_Avoid_: Latest plan, declared attendance

**ExpectedOccurrence**:
The current or historical planned reading of one OccurrenceRef, including applicable exceptions, openings and responsibilities.
_Avoid_: Declared attendance

**ObservedOccurrence**:
The reading of declarations and applicable inferences for one OccurrenceRef, with provenance and unresolved information visible.
_Avoid_: Mutable attendance fact, verified presence

**MonthlySummary**:
A monthly view of quantities and durations by person and activity, distinguishing declared and inferred values and identifying exclusions.
_Avoid_: Invoice, fully synchronized data

**Resolution**:
An explicit choice among a known set of conflicting declarations or actions, retaining the outcome and the history of discarded alternatives.
_Avoid_: Latest arrival wins, silent merge
