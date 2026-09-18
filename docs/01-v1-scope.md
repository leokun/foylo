# V1 scope

## Decided

The V1 and the data schema must be framed before any coding starts. The offline operation principle is retained.

## Confirmed V1 scope

The product owner confirmed this scope and the six validation scenarios on September 17, 2026 in [Confirm the V1 scope table and the six validation scenarios](https://github.com/leokun/foylo/issues/3). Delivery is iPhone-only, for one household shared by two adult accounts. Detailed business rules have adopted defaults; the [six journey acceptance criteria](18-v1-journey-acceptance.md) now specify required behavior, pending implementation and execution.

| Domain | Included in V1 |
| --- | --- |
| Household | One shared household, two adult accounts |
| Persons | Children and adults represented independently of accounts |
| Reference data | Places and activities |
| Planning | Weekly template, school calendars, public holidays, closures and exceptions |
| Responsibilities | Who drops off or picks up each person |
| Today | Activities, times, places, responsible adults and next action |
| Check-in | Start/end, end only, present/absent, quick adjustment |
| History | Traceable corrections and planned/observed separation |
| Summary | Monthly quantities and durations per person and activity, with Excel export |
| Support and feedback | Settings form, database-backed requests and restricted internal feature-prioritization list |

The support form was added to V1 by the product owner on September 18, 2026. See [support and feedback](19-support-and-feedback.md) for submission, privacy and acceptance rules. It supplements the original six journeys.

## Schedule pattern and one-action care recording

The latest product clarification supersedes the earlier interpretation of care automatically activating because no pick-up was recorded.

The example pattern defines school ending at 16:30 and a possible after-school-care start at 16:45. With no declaration, the normal assumption is collection at school dismissal: no care is recorded and no care duration is marked missing. This assumption is not a verified pick-up or an automatically authored declaration.

If the parent instead selects **Care exit** at 17:00, the application automatically records the interval's start at 16:45 from the pattern and its end at 17:00 from that action: 15 minutes of care. No arrival check-in, manual start or manual cancellation is required. Both boundaries are available in the recorded result, with their provenance distinguished: pattern-derived start and parent-declared end. Their representation as events or interval data is not yet chosen.

For morning care, the complementary rule is confirmed: if school starts at 08:30, selecting **Care entry** at 07:45 automatically records a 07:45-08:30 interval, giving 45 minutes. The start is parent-declared and the end is supplied by the school-start time in the pattern. No care-exit declaration is required for that interval. With no entry declaration, normal arrival at school is assumed, with no morning care recorded and no incomplete-care warning. The pattern-derived end is not an independently observed exit. Before 08:30, the application shows Care in progress, the declared entry at 07:45, the planned end at 08:30 and an expected duration of 45 minutes. Those 45 minutes are not presented as already elapsed or included in the calculated total before 08:30. At 08:30, they enter the calculated total with the end still identified as pattern-derived, without a manual exit declaration.

An optional explicit morning Care exit replaces the pattern-derived end while preserving history. With entry declared at 07:45, an exit declared at 08:10 gives 25 minutes based on two declared boundaries instead of the 45 minutes expected until 08:30. The interval is then completed and contributes 25 declared minutes to the calculated total.

Passing 16:45 without any action does not itself establish care attendance. The exit declaration establishes that care took place and provides the information needed to reconstruct its interval. This default is specific to the school/care pattern; it does not replace the separately confirmed lunch presumption.

Earlier proposals for mandatory generic child-pick-up declarations, automatic care activation in their absence, cross-day cancellation/recreation and future-session deletion workflows are not accepted. The later [V1 business decisions](12-v1-business-decisions.md) select dated revisions and define reconciliation and rescheduling; they do not reinstate automatic care activation or silent deletion.

## Confirmed planned/observed rules

The product owner confirmed the following rules on September 17, 2026 in [Planned/observed rules](https://github.com/leokun/foylo/issues/7):

- A planned school lunch without a check-in becomes presumed taken at the end of its planned time slot, unless an explicit declaration says otherwise. Before that boundary it remains planned and is excluded from the taken-meal total. No daily confirmation is required, and presumed meals remain distinguishable from explicitly declared attendance.
- When the parent declares an afternoon care exit, calculate the duration from its planned start and clearly identify that start as estimated. For a planned start at 16:30 and a declared pick-up at 17:48, the resulting 78 minutes use an estimated start, not a declared arrival.
- For morning care, a declared entry combines with the school-start time from the pattern to record both boundaries automatically; the end retains its pattern-derived provenance.
- An explicitly declared school-lunch absence replaces the presumption that the meal was taken. Contradictory declarations follow explicit conflict resolution, with no automatic winner.
- An arrival time declared afterwards replaces the estimated start in the duration calculation. The history is preserved. For an arrival later declared at 16:40 and a pick-up at 17:48, the duration becomes 68 minutes based on declared endpoints.
- For an explicitly declared care event, a missing required time or inconsistent timing is marked **Needs completion**. Its duration is excluded from the calculated total, and that total is explicitly marked incomplete. The confirmed planned-start fallback still applies when only the actual arrival declaration is missing: a valid estimated duration is not excluded merely because one boundary comes from the pattern.
- A check-in entered by mistake can be annulled while retaining its history. Annulment is not a silent deletion.
- Two check-ins of the same type for the same child and the same session are flagged as a suspected duplicate. Neither is automatically chosen as the one to retain. This is distinct from replaying the same technical operation.
- The monthly summary distinguishes declared and presumed meals, and durations based on declared endpoints from durations using a pattern-derived start or end. It also shows the number of sessions that still need completion.

- Resolve a suspected duplicate by explicitly choosing the correct check-in and annulling the other, retaining both histories. Until resolved, exclude the affected duration from the total and identify the summary as incomplete.
- After annulment, recalculate using the remaining information. For afternoon care, annulling a separately declared arrival restores the pattern-derived start while a care exit remains. Annulling the only morning care entry or afternoon care exit leaves no care declaration and restores the normal school arrival or pick-up assumption. Annulment of a lunch absence restores the presumed meal once the planned slot has ended, when the planned lunch remains and no other declaration overrides it. Before that boundary it returns to planned. A real absence must therefore remain explicitly declared.

A scheduled morning-care slot with no entry declaration follows the normal-school-arrival assumption; a scheduled afternoon-care slot with no exit declaration follows the normal-school-pick-up assumption. Neither is an incomplete care record. These rules do not create a declared presence or arrival from planning alone. Missing duration must not silently become zero. Concurrent corrections, competing duplicate resolutions and contradictory declarations follow the adopted [conflict rules](12-v1-business-decisions.md#concurrent-declarations-and-corrections).

## Confirmed planning-history principle

Weekly-template changes take effect from a chosen date without modifying earlier sessions or their calculations. A historical correction must be an explicit, traceable action. School-calendar updates also preserve past sessions and calculations; any retroactive correction requires an explicit, traceable action. Immutable planning revisions with effective dates are selected in the [V1 business decisions](12-v1-business-decisions.md#planning-versions-and-preservation-of-history).

Two time slots on the same day are distinct sessions, including when they concern the same activity. A one-off addition also has its own identity. Changing the time of a session without changing its day preserves its identity and the attachment of its check-ins and exceptions. This does not authorize silent recalculation of historical sessions.

Existing future ScheduleExceptions are preserved when the weekly template changes. If the new plan makes an exception incompatible, it is flagged for review rather than automatically deleted. For example, an exceptional 17:30 pick-up planned for next Thursday survives a template change effective Monday. When the new template removes the slot targeted by an existing exception, the affected session remains visible as To resolve. The parent explicitly chooses either to keep that session as a one-off activity or to cancel it, preserving history. No automatic deletion or silent choice is made. Calendar precedence and other incompatibilities follow the [V1 business decisions](12-v1-business-decisions.md#calendars-and-exceptions).

Moving a one-off activity to another day before any Punch has been recorded preserves the same occurrence identity, its information and the responsible adult. The date change remains traceable. For example, an appointment moved from Wednesday to Thursday remains the same appointment. If declared attendance reflects an appointment that actually took place on Wednesday, that appointment remains in history and a Thursday appointment is a new occurrence. If the Wednesday attendance declaration was a mistake, it must be explicitly annulled before moving the same appointment, preserving both the declaration and its annulment in history. The move does not change the date of the original declaration. Recurring-session moves and remaining declaration cases follow the [V1 business decisions](12-v1-business-decisions.md#occurrence-identity-and-moves).

## Confirmed quick actions and solo onboarding

The product owner confirmed Now as the primary time-entry action, with -5 minutes, -10 minutes and Choose a time as alternatives. Relative shortcuts use the action time, not the planned session time, and preserve the distinction between declared event time and recording time. They do not alter the existing care, history or conflict rules.

The product owner also confirmed that an adult can create a household, add children and planning, and use the application alone. Inviting the second adult is optional and can happen later; it is not an onboarding or daily-use gate.

## Confirmed weekly recurrence

The product owner confirmed one weekly template repeated identically each week, subject to the existing calendars, dated revisions and occurrence-specific exceptions. Alternating A/B weeks and multi-week rotation cycles are outside V1. Different children may still have different weekly schedules.

## Confirmed planning entry behavior

The product owner excluded copying a weekly template between children from V1. Configure each child's schedule independently.

When two activities for the same child overlap, show a non-blocking warning and allow saving. Preserve both occurrences and their identities; the warning does not merge, cancel or resolve either activity. A time overlap alone does not invalidate a declared duration or make a monthly total incomplete. Other calendar, timing and conflict rules still apply independently.

## Confirmed activity presets and initial trial

The product owner confirmed School, School lunch and Care as activity presets ready to configure, with custom activities also available in V1. For a custom activity, the product owner confirmed a choice of tracking mode at creation: Planning only (default), Presence/absence, or Start/end for actual duration. Planning only requires no attendance declaration; absence of a Punch is not missing attendance. Presence/absence does not inherit school-lunch presumption, and custom Start/end does not inherit the school/care pattern boundary fallback. Existing explicit-record completeness rules still apply.

Presets do not fabricate schedules or attendance: the adult supplies the relevant planning configuration. Existing school, lunch and care rules remain authoritative.

The first trial is limited to the product owner's household. Additional pilot households are not part of the initial trial. This does not remove household isolation requirements or replace acceptance testing.

## Confirmed onboarding and summary delivery

The product owner confirmed a short initial setup: create the household, add a child, then configure that child's weekly schedule. Allow leaving and resuming the setup later without repeating completed steps. Preserve saved progress; an unfinished setup does not require inviting another adult.

The monthly summary is available in the application and as an Excel export (.xlsx). The product owner's subsequent request for an Excel recap supersedes the earlier in-app-only decision. Keep actual recorded times and contract-retained times in separate columns; never relabel retained times as observations. Other export formats are not selected.

The product owner requires two export packaging options: separate files per child, or one file containing all children. Both use the same selected month and calculation snapshot. Each child has an [First name] École worksheet plus a separate worksheet for each of that child's caregivers. Use a recognizable title such as [First name] [Caregiver name], with the full child/caregiver identities inside the report. The product owner explicitly selected one worksheet per child per caregiver, superseding the earlier single Assistante maternelle worksheet per child. A child with two caregivers therefore has three worksheets including École. Separate child files retain the same applicable worksheets and calculations as the combined workbook.

The École worksheet contains school care (morning and afternoon as applicable) and school lunch, with separate quantities/durations and subtotals. Each caregiver worksheet contains that arrangement's relevant daily planned, actual and retained times, normal hours, supplementary hours, meals and snacks (goûters), with subtotals and monthly totals. Keep actual elapsed duration separate from retained duration and keep hours separate from food quantities; no total combines unlike units. Do not combine school lunch with nanny meals into an unexplained single quantity. Monetary amounts remain outside V1.

Preserve recognizable first names in worksheet titles and disambiguate identical names or sanitized/truncated title collisions without mixing children. Exact Excel title constraints and safe text handling must be verified during export implementation. The displayed report identifies its child independently of the worksheet title.

The product owner confirmed planning nanny meals and snacks in the weekly template or the nanny's typical-day definition, allowing different selections by scheduled day. Food is explicit planning data, not inferred solely from care duration. This requirement does not imply a new reusable-template library or copying plans between children. Preserve dated planning versions and occurrence-specific changes.

Whether early/late supplementary time can be offset by a shorter interval elsewhere that day remains unconfirmed. The product owner does not know of a compensatory rule and explicitly left it to confirmation. Do not silently introduce compensation, but do not present its prohibition as a confirmed contractual rule either. Resolve this with the applicable contract before treating the normal/supplementary split as accepted.

The product owner confirmed that nanny meal/snack quantities count only items supplied by the caregiver. Items brought by parents are excluded from these counts. In the weekly or typical-day setup, label the meal/snack selections explicitly as caregiver-supplied, and apply the same meaning to per-day corrections, app subtotals and Excel totals. These counts do not represent all food consumed by the child.

The product owner confirmed nanny meal/snack counting from the weekly or typical-day plan, removing items affected by child or caregiver absence and allowing occurrence-specific manual correction. Keep plan-derived quantities distinguishable from explicitly confirmed or corrected quantities. Do not count an unplanned item solely because care duration spans a meal. For a half-day change, the product owner confirmed that the parent specifies meal/snack exclusions either in advance while editing the dated day, or when collecting the child or recording the change: offer Remove meal and Remove snack as independent choices, allowing either, both or neither. Do not require configured meal/snack times or infer those exclusions from collection time. These choices adjust only that occurrence, preserving the weekly/typical-day plan and traceable corrections. Advance exclusions persist through later synchronization and collection; opening or recording departure must not reset them or deduct the same item twice. The parent may explicitly revise the selection later. Planned food remains the default for that day unless explicitly excluded or overridden by a full-day absence; label the resulting quantities as plan-derived rather than independently observed consumption. Future food stays planned. A display time must not fabricate a declaration or trigger a guessed half-day deduction.

For caregiver duration rows, include date, child, activity, actual arrival, actual departure, retained start, retained end, actual elapsed duration, counted duration and status. Include enough rule context to explain the retained reading, including the applicable increment or exact-minute mode and contract version. Display time and duration column labels in French. For arrival 08:47 and departure 17:25 under the example 30-minute contract, show actual duration 8 h 38 separately from retained 09:00-17:30 and counted duration 8 h 30.

The workbook reflects the selected month and person/activity filters, using the same reading as the in-app summary. Keep actual and counted totals separate. Missing or conflicting times remain explicitly identified rather than exported as zero or fabricated timestamps; only eligible readings contribute to their respective totals. Include generation time, household timezone and synchronization freshness so pending local readings are not mistaken for fully synchronized results. The export is a snapshot of the current traceable reading, not a replacement for correction history or an invoice.

## Confirmed interface language and weekly view

The product owner confirmed a French-only user interface for V1. Repository documentation and code remain in English under project conventions; this does not determine the user-facing language.

Provide a weekly planning view alongside Today to inspect upcoming days and make planning changes. Existing effective-date and occurrence-specific exception rules apply: editing one dated session must not silently change the weekly template or historical sessions.

## Confirmed edit scope and immediate annulment

When editing the time of a recurring occurrence, offer Only this day and From this date, defaulting to Only this day. The first creates a dated exception; the second creates an effective-dated planning revision. Existing exceptions and historical readings retain the protections already specified. Retroactive changes still require explicit historical correction rather than bypassing those rules through this selector.

After recording a Punch, offer an immediate Undo action. It appends a traceable annulment rather than deleting the declaration. Preserve the original actor and recording history and apply the existing recalculation and concurrency rules. If the declaration is pending synchronization, keep both actions durably ordered for replay; do not report server acceptance before acknowledgement.

## Confirmed care setup and conflict entry point

Configure morning care and afternoon care separately for each child, including their respective habitual days and times. Preserve the confirmed one-action rules: morning Care entry uses the planned school-start boundary; afternoon Care exit uses the planned care-start boundary. Separate configuration does not activate attendance without a declaration.

Today includes an A verifier entry point when attendance declarations conflict. Opening it shows the competing declarations and lets the adult explicitly choose the correct one, preserving the discarded declaration through traceable annulment or resolution. No fact is chosen automatically. Existing rules for affected totals, newly arriving conflicts and competing resolutions still apply. The user-facing French label is À vérifier.

## Confirmed school-care counting options

The product owner confirmed that school care also supports Exact minutes or Increments of 5, 10, 15 or 30 minutes, configured for the applicable establishment/care arrangement. Preserve the selected policy through effective-dated revisions and expose calculated source duration separately from retained counted duration in the app and École export worksheet.

For school care and nanny/nursery arrangements, the product owner additionally requires an explicit rounding direction when Increments is selected: Up or Down. Exact minutes does not apply rounding. Keep the direction with the increment and effective-dated policy; it must not be silently hard-coded. Up means the next grid boundary and Down the preceding boundary for the value being rounded; values already on a boundary remain unchanged.

This choice does not change the confirmed one-action school/care behavior or boundary provenance: morning entry can use the school-start boundary and afternoon exit can use the planned care-start boundary. Those calculated durations remain estimated when a boundary is pattern-derived, even if a counting rule then rounds them. Never relabel them as fully observed actual durations.

The product owner confirmed clock-time rounding by default for both school care and nanny/nursery arrangements, with no duration-versus-time selector in V1. Round the applicable arrival/departure clock boundaries, then derive counted duration from the retained boundaries; do not round the elapsed duration itself. Keep school-care pattern-derived boundaries distinguishable from declared ones. Nanny tolerance windows and baseline guarantees do not automatically apply to school care. The product owner confirmed independent directions for arrival and departure: Down for arrival and Up for departure by default, each configurable for the applicable contract or establishment. Pricing remains outside V1.

## Confirmed multiple concurrent caregivers

The product owner confirmed that one child may attend multiple nannies or nurseries in the same week. Required example: caregiver A on Monday, Tuesday, Thursday and Friday, and caregiver B on Wednesday. This is concurrent supported planning, not a historical replacement or an alternating-week pattern.

Each child's care arrangement has its own provider identity, assigned weekdays and slots, applicable calendar/leave, expected windows, contract counting policy, normal/supplementary-hour rules, absence defaults, and planned caregiver-supplied meals/snacks. Editing one arrangement must not silently change the other. Today, weekly planning, recording and history identify the actual arrangement for every occurrence. Provider identity does not grant an account or external access.

The product owner declined a dedicated one-day caregiver replacement feature for V1 because last-minute replacements are too rare in their usage to justify dedicated functionality. Do not add an action that swaps the caregiver and automatically substitutes their contract rules for that day. This does not remove the confirmed concurrent weekly arrangements or ordinary dated planning exceptions.

Keep each declaration and its planning/contract basis attached to its original arrangement across future changes. A caregiver's leave affects only their applicable sessions, not the child's other caregiver. Keep app and export subtotals attributable to each arrangement, with original recorded times and retained times distinguishable. The previously selected per-child or combined workbook packaging remains. Each child/caregiver pair has its own worksheet and totals; do not combine multiple caregivers within one worksheet. The conceptual model must explicitly support these concurrent arrangements before persistence contracts are frozen.

## Confirmed caregiver contract windows and counted duration

The product owner clarified that nanny and nursery duration accounting follows the applicable contract, not simply elapsed time between declarations. Preserve exact recorded arrival/departure times and distinguish them from the retained boundaries and counted duration. This supersedes the earlier interpretation that every nanny duration directly equals elapsed time between punches. Monetary pricing remains outside V1.

Example contract: baseline 09:00-17:00, accepted arrival window 08:45-09:00 and departure window 16:45-17:00. Arrival and departure within those respective windows count as 09:00-17:00, or 8 hours. For arrival recorded at 08:47 and departure at 17:25, the product owner's example retains 09:00-17:30, or 8 hours 30 minutes, under that contract's rounding rule. Do not replace the original declarations with retained contract times.

Configure supplementary-time accounting when setting up the nanny or nursery: Exact minutes, or Increments with a choice of 5, 10, 15 or 30 minutes. This explicit product decision replaces the earlier unresolved choice between fixed 15- and 30-minute rules. Scope the setting to the applicable child's care arrangement and preserve effective-dated versions for historical interpretation and offline declarations. No monetary rate is introduced.

Keep the confirmed baseline and tolerance behavior. In the example, arrival within 08:45-09:00 or the parent's late arrival at 09:10 retains the 09:00 counted start. Departure within 16:45-17:00 retains 17:00. Outside tolerance, Exact minutes retains the actual relevant boundary. The earlier examples use downward arrival rounding and upward departure rounding on the selected wall-clock grid. The subsequent explicit requirement for configurable Up/Down rounding supersedes treating those directions as mandatory for every contract. The product owner subsequently confirmed rounding clock-time boundaries, not durations, without offering a basis selector in V1. Apply contract tolerance/baseline rules before the applicable out-of-window rounding. Arrival and departure have independent direction settings, confirmed by the product owner: Down at arrival and Up at departure by default, both configurable. A 5-minute grid uses :00/:05/:10/etc., 10 minutes uses :00/:10/:20/etc., 15 minutes uses :00/:15/:30/:45, and 30 minutes uses :00/:30. Exact grid boundaries stay unchanged. The grid is not anchored to the contractual start.

For example, arrival 08:47 and departure 17:25 count as 09:00-17:25 in Exact minutes mode (8 hours 25 minutes), and 09:00-17:30 with a 30-minute increment (8 hours 30 minutes). Arrival 08:35 retains 08:30 with a 30-minute increment; arrival 08:10 retains 08:00, with no one-increment cap. The earlier early-departure example is also preserved: under the 30-minute rule, arrival 09:10 and departure 16:20 retain 09:00-16:30, or 7 hours 30 minutes, without imposing an eight-hour minimum. Keep original declarations in every case.

Show expected windows in Today, the weekly view and the recording context. History and summary must explain recorded times, retained boundaries, counted duration and the applied contract rule. Missing required declarations remain incomplete rather than fabricated; the school/care pattern's automatic boundary fallback does not apply to nanny care. For a declared nanny/nursery absence, choose the retained hours case by case, prefilled from the applicable contract default. The product owner explicitly excludes justification tracking: no reason, proof status, attachment or medical detail is requested for this choice. Confirm retained hours explicitly rather than inferring them from evidence. An absence remains an absence even if hours are retained: create no actual arrival/departure or actual care duration. Keep the retained absence hours identifiable in the app and Excel totals, with traceable changes and the applicable contract version.

This is a time-accounting requirement, not an assertion about contractual or statutory billing obligations. The conceptual model and monthly summary must be reconciled with the distinction between elapsed and contract-counted duration before persistence contracts are frozen.

## Confirmed weekday selection and responsibilities

Weekly setup allows selecting any subset of weekdays and entering their common times once. In particular, care may be configured for only two days, such as Tuesday and Thursday, with different selections for morning care, afternoon care and each child. Saving does not add unselected days. This is a setup convenience and does not introduce alternating weeks or copying another child's plan.

Drop-off and pick-up have separate optional responsible adults. Either can be left unassigned, and different adults can hold the two responsibilities. Assignments do not create attendance facts or grant account access.

## Confirmed Today layout

The product owner confirmed a shared Today view showing all children, with activities ordered chronologically and an optional filter by child. Filtering changes only the displayed subset, not assignments, planning or recorded facts.

## Confirmed calendar geography and one-off form

The product owner confirmed France-only school calendars for V1. Configure the household school-calendar reference with institution-specific overrides and explicit closures. This geographical scope does not claim that every French territory or school year is already covered by the selected data source. Verify available academy/territory coverage and expose unsupported or missing coverage as unknown; never substitute metropolitan zones silently.

The product owner confirmed academy selection with automatic school-zone mapping, using the applicable school-year reference rather than requiring knowledge of A/B/C. Keep institution-specific overrides and separate territory handling.

The product owner confirmed two independent activity settings: Closed during school vacations and Closed on public holidays. For example, nanny care can remain open during school vacations while closing on public holidays. These settings configure general calendar policy; explicit dated openings and closures retain their established precedence.

Public holidays are also included in the V1 calendar calculation. Import the applicable official French public-holiday calendar separately from school vacations, preserving territory, year and source. Apply each activity's configured closure policy and explicit institution openings; a public holiday does not prove that every nanny or care service is closed. A day that is both a public holiday and a school vacation is excluded once from the planned attendance count, without double subtraction. Historical revisions and unknown-coverage rules apply to both imports.

The product owner explicitly requires importing the state's school-holiday data for the zones in V1, rather than entering holiday periods by hand. Use the Ministry's [official school-calendar dataset](https://www.data.gouv.fr/datasets/le-calendrier-scolaire) as the import source, with the [published official calendar](https://www.education.gouv.fr/calendrier-scolaire-toutes-les-dates-des-cours-et-des-vacances-100148) as the interpretation reference. Import and retain validated data server-side, then synchronize the relevant calendar to devices for offline use. No importer has been implemented or run as part of this decision.

The initial household is in Brittany. Rennes is in Zone B according to the official calendar; use Rennes/Zone B as the proposed initial setup value, while keeping the actual school calendar selectable. Do not infer every establishment's opening policy from the household location.

The import must preserve source, school year, academy/territory, population, source dates and retrieval time. Validate complete responses before publishing a revision; technical retries must not duplicate periods. Refresh failure or invalid data retains the last accepted version. Missing year/territory coverage stays unknown. Source updates must preserve local institution exceptions and past planning versions. Use the [calendar research](research/school-calendars.md) for departure/return boundaries, summer markers and timezone interpretation; raw timestamp ranges are not sufficient closure rules. School-holiday imports do not automatically close unrelated daycare or custom activities.

The product owner confirmed a simple one-off activity form: Person, Activity, date, times, Place and an optional responsible adult. Show the resulting occurrence in Today alongside recurring activities. Preserve the existing stable identity, calendar precedence and traceable rescheduling rules. An unassigned responsibility remains visible and does not block saving or grant anyone access.

## Confirmed closures and caregiver unavailability

V1 must let either adult record school closure or nanny unavailability, including training, that overrides the usual weekly plan on the affected dates. These are dated calendar exceptions, not edits to the recurring template and not declarations of a child's absence.

Choose a date or date range, a reason, and the affected service/activity calendar. The product owner also confirmed partial-day unavailability: morning, afternoon or a precise time range. Morning/afternoon are shortcuts to explicit start and end times shown before saving, not hidden universal cut-offs. Store and interpret those boundaries in the household timezone. Show the impacted children and sessions before saving. Apply a shared service closure to all occurrences using that calendar, including multiple children, without requiring duplicate entries. Do not infer that every activity at the same Place is closed: school, lunch and care must be included explicitly in the affected scope or share the configured closure calendar. Unrelated services remain unchanged.

For a partial closure, show its overlap with the planned sessions. A fully covered session is closed; a partially overlapping session stays visible as To resolve, displayed as À adapter in this interaction, until its hours or cancellation are explicitly decided. The product owner confirmed this behavior for care planned from 08:00 to 18:00 with unavailability from 14:00 to 16:00: offer explicit time editing or cancellation, without automatic splitting. Sessions outside the unavailable interval remain unchanged. Do not silently split an occurrence, shorten a declared interval or assume availability on either side of the closure. Until resolved, affected reminders and automatic meal presumption are suppressed.

Retain the usual template for subsequent open days. In Today and the weekly view, keep a visible closure/unavailability indication and reason while excluding affected sessions from the active plan, associated reminders and automatic meal presumption. Do not create an observed absence or erase an existing Punch. Conflicting declarations or incompatible one-off additions remain visible for explicit review under the existing rules. Accepted historical calculations require an explicit historical correction before a backdated closure changes them.

## Confirmed caregiver leave visibility

The product owner clarified that the requested nanny absence record means the nanny's leave days (conges). Record caregiver leave explicitly and separately from child absence and caregiver training. Use the existing dated caregiver-unavailability calendar flow with the applicable service scope, identifying all affected children's sessions without duplicate entries. Other unavailability continues to follow the previously confirmed closure rules.

The product owner clarified the immediate requirement: display Congé on the affected dates in the caregiver planning. Show this label in Today and the weekly view, with caregiver context explicit, and preserve the leave indication in each affected child's Assistante maternelle worksheet. No paid/unpaid classification or additional justification field is required by this request. Keep leave rows visible even when no care session contributes to attendance totals. Do not label leave as a child's absence or create a child absence Punch.

The weekly template remains intact and existing attendance remains traceable. Apply closure and partial-overlap rules. Remove affected planned meals/snacks from counted quantities under the confirmed food rule, with explicit corrections still available. A leave entry does not establish actual care or determine contractual remuneration or counted hours. No zero-hour default has been confirmed for caregiver leave; do not apply one or copy the child-absence default. Leave classification and any contract-retained-hour treatment remain to clarify. Monetary pricing remains outside V1.

## Confirmed whole-day and half-day child absence

The product owner confirmed a single action to declare a child's absence for a whole day or half-day. Show the date, explicit period boundaries and affected activities before confirmation; allow excluding activities from the selection. Do not require a medical reason or collect health details. This action concerns the selected child, not an institution closure or another child's attendance.

For each selected occurrence fully covered by the absence period, record an explicit absence where attendance tracking supports it, preserving occurrence identity and history. A lunch absence overrides meal presumption. For planning-only activities, offer an explicit planned cancellation instead, clearly labelled in the confirmation; do not create attendance tracking implicitly.

If a session only partly overlaps the selected half-day, flag it for explicit adjustment rather than declaring the entire session absent or splitting it automatically. Existing attendance is never silently overwritten: contradictory declarations follow conflict resolution. The weekly template remains unchanged. Preview the complete change set and report pending, accepted or rejected state without claiming that an unaccepted action has synchronized.

## Validation still pending

 Concurrent facts, historical plan changes and calendar precedence have adopted defaults in the [V1 business decisions](12-v1-business-decisions.md); their execution and browser verification remain pending.

## Confirmed V1 validation scenarios

These are required journeys, not completed acceptance tests. Their [detailed acceptance criteria](18-v1-journey-acceptance.md) are now specified.

1. Set up two children with different places and weekly templates.
2. Automatically exclude periods when the school is closed.
3. Change the responsible adult on one Thursday without affecting the following Thursdays.
4. Record a pick-up offline, then find the fact on the second phone.
5. Correct 17:48 to 17:43 while keeping the history of the correction.
6. Compare planned and declared over a month, with missing data identifiable.

## Rejected/deferred

Explicit rounds, navigation, pricing, Android, web, Watch, multiple households, sophisticated external permissions and rich push remain outside V1. See [future scope](09-future-scope.md). Deferral does not commit any later delivery.
