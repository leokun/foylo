# V1 business decisions

Status: adopted V1 defaults under the product owner's instruction to resolve the remaining business choices without individual confirmation. These defaults complete the current planning and planned/observed discussion. They are design decisions, not implemented or tested behavior. Earlier individually confirmed rules remain in force.

## Occurrence identity and moves

- Each recurring slot has a stable identity, independent of its times and planning version. A recurring occurrence is identified conceptually by that slot, the person and its original local date. Two same-day slots remain distinct. A one-off activity has its own stable identity.
- Moving one recurring session from Wednesday to Thursday is a dated exception targeting the original occurrence. It retains its identity, information and responsible adult; other Wednesdays remain unchanged. The original date stays part of its identity and history, not its current display date.
- Moving a one-off activity also retains its identity. A target-day overlap is flagged for review; it does not merge two sessions.
- A session with actual attendance stays in history. A later appointment is another occurrence. A mistaken declaration must be explicitly annulled before rescheduling; the original declaration and its dates remain in history. These rules apply to recurring and one-off sessions alike.
- Existing declarations of absence are also retained on their original session. Replanning a missed session creates a new occurrence, unless the absence was mistaken and explicitly annulled first.

## Planning versions and preservation of history

- Rules, calendars and exceptions have immutable revisions with an effective date and a traceable author/source. New revisions apply from a chosen date. An ordinary future change never rewrites earlier calculations.
- Existing recurring slot identities survive time changes. Adding another slot creates a distinct identity; removing a slot does not erase its history.
- Retain the planning revisions used to derive historical boundaries. A late or offline declaration retains its original planning basis. If reconciliation reveals an incompatible version, flag it for explicit resolution rather than silently changing its inferred duration.
- Historical corrections append a traceable revision with a reason. Keep the previous reading available. The current corrected reading may then change totals explicitly.
- Future exceptions survive template changes. If a removed slot has an exception, show To resolve until the parent explicitly keeps it as a one-off session or cancels it. Retention preserves its existing identity and references.
- Removing an ordinary future slot with no exception or declaration removes it from the computed future plan, while keeping the rule history. A session with declarations remains visible and cannot be silently erased by a planning edit.
- Concurrent planning edits target a known revision. An edit based on an outdated revision is retained as a conflicting proposal for review; it does not overwrite the accepted revision. Do not merge fields automatically.

## Calendars and exceptions

Calendars attach to the relevant activity/place. School holidays do not automatically close a nanny or daycare activity that does not follow that calendar.

Apply these rules in order:

1. An explicit personal cancellation removes the planned attendance for that person. It does not create an observed absence or erase declarations.
2. A dated, explicit institution opening or closure overrides general holiday defaults for that activity. Conflicting explicit institution records require resolution rather than automatic priority.
3. Otherwise apply the school/public-holiday closure policy configured for that activity, then its weekly schedule.
4. A personal time change or addition applies only within a known compatible opening. It cannot silently reopen a closed institution. An incompatible request stays visible as To resolve until its plan or the institution information is explicitly corrected.

Missing calendar coverage remains unknown. Keep the latest known cached coverage on refresh failure. Show affected planned sessions as unverified and exclude them from automatic meal presumption until coverage or an explicit dated opening is supplied. Do not assume that missing data means open or closed. Valid explicit attendance remains visible even when the calendar disagrees; flag that disagreement without fabricating or discarding facts.

## Concurrent declarations and corrections

- A retry with the same operation identity and content yields the same result. The same identity with different content is rejected as inconsistent. Two independently identified declarations remain distinct, even at identical times.
- Suspected duplicate check-ins have no automatic winner. The parent explicitly retains the correct declaration and annuls the other, keeping history.
- Two competing corrections, a correction competing with an annulment, or contradictory present/absent declarations remain unresolved. Exclude the affected duration or meal quantity from aggregates and mark the summary incomplete; never fall back to meal presumption over a known conflict.
- A resolution identifies all competing facts and the selected outcome. If new conflicting facts arrive, reopen the conflict. Competing resolutions also require explicit resolution. An ordinary correction made after an accepted resolution follows the normal traceable correction flow.
- The same complete set of facts must produce the same reading on every device, regardless of arrival order. Neither the earliest device time nor the latest server arrival chooses the business truth.

## Time and offline interpretation

- Keep the declared event time, device recording time and server receipt time distinct. Event time determines attendance and duration; recording and receipt times provide audit information, not conflict priority.
- Use one explicit household reference time zone, defaulting to Europe/Paris for the initial V1. Travel or a phone time-zone change does not move household sessions. A household zone change applies prospectively through a planning revision.
- Store declared instants with sufficient original zone/offset context to explain them. Rules use local times. Durations use elapsed time between valid instants; monthly grouping uses the session start date in the household zone.
- Cross-midnight sessions require an explicit next-day end. Never infer an overnight session merely because its end precedes its start. Ambiguous or nonexistent daylight-saving times require a valid explicit choice before duration calculation.
- Offline displays use device time provisionally for in-progress care and meal transitions. On reconnect, reconcile against server time without rewriting declared event times. A clock-related provisional summary may change and must retain its provenance.
- The already confirmed morning rules remain: expected duration while care is in progress, inclusion at the planned end, or earlier completion using an explicitly declared exit. Annulment of that exit restores the pattern-derived end and the corresponding in-progress/completed state.

## Validation matrix

These are required checks. The [prototype verification report](13-prototype-verification.md) records executed fixture examples and pending integration/browser coverage; it does not establish application acceptance.

| Case | Required result |
| --- | --- |
| Move next Wednesday's recurring session to Thursday | Same identity and responsible adult; other Wednesdays unchanged |
| Move onto another session of the same activity | Distinct identities, overlap visible, no automatic merge |
| Actual Wednesday attendance followed by Thursday appointment | Wednesday history retained; Thursday is a new occurrence |
| Mistaken Wednesday attendance annulled before move | Same appointment moved; original fact and annulment retained |
| Template changes Monday; Thursday has a personal exception | Exception retained; incompatibility flagged |
| Removed slot has an exception | To resolve; explicit one-off retention or cancellation |
| Late offline care exit uses an older applicable pattern | Original planning basis retained; incompatible reconciliation flagged |
| Institution closed but personal attendance planned | No silent reopening; planning exception needs resolution |
| Institution explicitly open during general holiday | Dated opening governs that activity |
| Missing calendar coverage and no lunch declaration | No presumed meal; unverified coverage visible |
| Two opposing corrections arrive in reversed order | Same unresolved reading; affected total excluded |
| Presence and absence conflict at lunch | Neither declared meal nor presumed meal counted until resolution |
| Repeated upload of one operation | One accepted fact; content mismatch rejected |
| Morning entry 07:45, planned end 08:30, clock 08:29 then 08:30 | Expected 45 minutes becomes 45 estimated minutes in the total at 08:30 |
| Morning exit declared at 08:10 | 25 declared minutes; no second interval at 08:30 |
| Exceptional morning exit annulled | Planned end restored, with state based on the current clock |

## Delivery boundary

The temporary prototype has been extended with 20 executable fixture scenarios, documented in the [verification report](13-prototype-verification.md). Their outputs support the investigated rules without validating the complete integrated model. Browser verification, real synchronization and the broader concurrency-resolution workflow remain pending. No production code, schema, infrastructure, commit, push or GitHub issue closure is implied by this document.

The [conceptual model](02-domain-model.md) is now frozen for V1. Technical foundation selection, access/privacy/notification requirements and full V1 journey acceptance criteria are the next specification work. The product owner's mandate permits reasonable defaults without another per-case interview; it does not turn an unverified capability into evidence or authorize starting implementation.
