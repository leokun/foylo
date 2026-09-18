# V1 journey acceptance criteria

Status: adopted acceptance specification for the six confirmed V1 journeys. All journeys remain pending end-to-end acceptance. Existing domain and simulator experiments are partial evidence, not a pass for these criteria.

## Shared test conditions

Use one household, two adult accounts A and B, and two children with distinct activities and places. Use the household timezone, initially Europe/Paris. Record the tested build, devices, accounts, initial planning revisions, action sequence and resulting histories. Use synthetic data.

Every journey must preserve the distinction between planned, declared, presumed, estimated, incomplete and conflicting information. Display synchronization status separately from business completeness: a locally complete interval can still await upload. Responsibilities grant no extra access. Denied notification permission must not block any journey.

For offline actions, require a valid local access grant. Expiry locks access and preserves the encrypted queue; confirmed typed membership revocation applies the cleanup policy. A pending or rejected operation must never appear server-accepted. Replays use the original operation identity, actor and planning basis. Use the [access policy](17-access-and-privacy.md) for the full failure matrix and [notification policy](06-notifications.md) for N01-N10.

The V1 user interface is in French. Both Today and a weekly planning view are available; business provenance, synchronization status and validation messages must be understandable in that language.

## J01: Set up two children with different plans

Given A has created a household without inviting B yet:

- Initial setup follows create household, add child, then configure the child's week. Leave after a completed step and reopen: saved progress remains, completed entities are not duplicated, and setup can resume without an invitation.
- Creating a custom activity defaults to Planning only and offers Presence/absence or Start/end tracking. Planning-only occurrences generate no missing-attendance prompt; custom presence is never automatically presumed like school lunch; custom durations do not silently borrow school/care boundaries.
- Select only Tuesday and Thursday for care and enter common times once: both selected weekdays receive the schedule and no other day does. Morning/afternoon selections and different children remain independent.
- Set different adults for drop-off and pick-up, then clear either assignment: saving succeeds, the other assignment remains unchanged, and no attendance or access permission is created.
- For the same child, configure caregiver A on Monday/Tuesday/Thursday/Friday and caregiver B on Wednesday. Use different counting increments, food selections and leave calendars. Verify that Today, weekly planning and recording select the correct arrangement, changes and leave stay scoped, and historical facts retain the original provider/contract basis. No caregiver account access is implied.
- Configure nanny arrival expected at 08:45-09:00 and departure expected at 16:45-17:00. Today, the weekly view and recording context preserve both windows. Different children and dated exceptions remain independent.
- Configure morning and afternoon care independently for each child, with separate days and times. The resulting plan uses the correct school/care boundaries without generating attendance from configuration alone.
- Setup offers School, School lunch and Care presets plus a custom activity option. Selecting a preset alone creates no attendance declaration or assumed schedule; configuration remains explicit.
- A creates two distinct Persons, places and activities, and assigns each child a weekly template with a chosen effective date. Account identity remains separate from child identity. Setup, Today and recording remain available without sending an invitation.
- Today shows each child's own activities, times, place, responsible adult and next available action. The default view combines all children in chronological order; selecting a child filters the list, and clearing the filter restores the combined view without changing data. Changing the first child's plan does not change the second child's plan.
- Create a one-off activity with Person, Activity, date, times and Place, leaving the responsible adult empty. Saving succeeds and Today shows the distinct occurrence as unassigned; assigning an adult later does not create another occurrence.
- Each child's weekly template repeats identically on successive weeks unless a dated revision, calendar or explicit exception applies. No A/B rotation configuration is offered in V1.
- Configure children independently; V1 offers no cross-child template copy. When activities for the same child overlap, warn without blocking saving and retain both occurrences. An overlap alone does not create an incomplete total; other validation rules still apply.
- Two slots for the same activity on the same day remain distinct occurrences. Reopening or retrying the setup does not duplicate an already accepted operation.
- A can invite B later. After B accepts the targeted invitation, B sees the accepted shared plan after synchronization. An unrelated account cannot read it. An account cannot join a second active household.
- Editing a template creates a new dated revision. Past sessions and their calculations remain unchanged. Existing future exceptions survive, with incompatible ones marked To resolve.

Pass evidence: complete setup and record an action with A alone, then invite B and inspect both daily plans on both accounts, retained revision history and one incompatible future exception. Exercise access denial separately from merely hiding a screen.

## J02: Exclude school closures automatically

Given school has a weekly template and verified French calendar coverage, while a daycare activity uses a different calendar:

- V1 offers French school calendars only, with the household reference and institution-specific overrides. Unsupported territory/year coverage stays explicitly unknown rather than being silently mapped to another zone.

- Selecting Rennes maps to Zone B using the relevant school-year reference. Institution overrides remain available; unsupported territories are not forced into A/B/C.
- Configure Closed during school vacations and Closed on public holidays independently for an activity. Verify all four combinations, including nanny care open during vacations but closed on public holidays, and verify that explicit dated opening/closure rules still take precedence.
- Import the applicable official public holidays independently of school vacations. A holiday already inside a vacation period does not double-reduce planned quantities. An explicitly open care service stays open under its own calendar; unavailable holiday coverage remains unknown.
- Import official state school-holiday records for A/B/C, retaining academy, population, school-year and source distinctions. Verify Rennes/Zone B for the initial household, successful retry without duplication, offline availability and a failed or malformed refresh retaining the previous accepted version.
- Exercise unknown future-year coverage, an upstream correction with preserved history and institution overrides, holiday departure after lessons, the return morning, daylight-saving conversion and a summer-start marker. Do not infer a complete summer interval from a missing return date.
- A school closure excludes the affected school sessions from the active plan and automatic lunch presumption. The unrelated daycare plan stays available.
- Record a school closure and a separate nanny training day or date range. Preview the affected children and sessions; one shared-calendar entry applies to all relevant children, while unrelated activities remain unchanged. The weekly template resumes on the next open scheduled day.
- Record morning-only, afternoon-only and precise-hour unavailability with explicit visible boundaries. Fully covered sessions close; partial overlaps remain To resolve (À adapter in this interaction), offering time editing or cancellation without silent splitting or recalculation of declarations; sessions outside the interval stay unchanged. Suppress affected reminders and automatic meal presumption until resolution.
- Record the nanny's leave with the Congé planning label, including a shared calendar affecting two children. Distinguish leave from child absence and caregiver training in Today, the weekly view and both relevant Assistante maternelle export sheets. Keep leave rows visible even when no attendance contributes to totals. Remove affected planned meals/snacks; do not create child absence declarations or silently apply a zero-hour or child-absence contractual default. Contract-retained leave hours remain pending confirmation.
- Today and the weekly view show the closure reason. Closed sessions create neither attendance nor child absence, automatic meals or responsibility reminders. School closure does not silently cancel separately configured lunch or care outside its declared scope.
- An explicit institution opening overrides a general holiday closure. An explicit personal cancellation still removes that person's planned attendance without creating an observed absence.
- A personal addition cannot silently reopen a closed institution. Contradictory institution records and incompatible exceptions remain visible for resolution.
- Missing coverage is shown as unknown, not open or closed. A failed refresh preserves the last known coverage; uncovered lunches never become presumed meals.
- A newly imported future calendar revision does not rewrite historical sessions. Existing declarations that disagree with a closure remain visible with the disagreement flagged.
- Cancelled or unverified sessions do not produce responsibility reminders.

Pass evidence: exercise school closure and nanny training across shared and unrelated calendars; compare school and daycare on the same closure day, an explicit opening, an uncovered date and a historical declaration retained after an update.

## J03: Reassign one Thursday

Given A is responsible for a recurring Thursday pick-up:

- From the weekly planning view, A selects one dated occurrence and assigns B. The selected Thursday shows B; preceding and following Thursdays retain their existing assignments.
- The exception preserves the occurrence identity and records its author and revision. No check-in is created by the assignment change.
- B receives the changed assignment after synchronization. If reminders are enabled and still eligible, A's reminder is removed and B's is scheduled under the notification policy.
- Editing a recurring occurrence time offers Only this day (default) or From this date. Verify that the first changes only the selected occurrence and the second applies a dated template revision without rewriting earlier sessions or silently dropping exceptions.
- A later weekly-template revision preserves this exception. Removing its slot leaves the affected occurrence To resolve until explicitly retained as a one-off session or cancelled.
- Concurrent edits based on the same old revision do not silently overwrite one another. Retain the conflicting proposal for explicit resolution; do not claim that both phones agree before convergence.

Pass evidence: inspect three consecutive Thursdays, the exception history, reminder reconciliation and one stale concurrent edit.

## J04: Record a pick-up offline and recover it on the other phone

Use the confirmed one-action care case: the pattern starts afternoon care at 16:45 and A selects Care exit at 17:00 while offline. This journey does not make ordinary school pick-up a mandatory declaration.

- Offer Now as the primary action, with -5 minutes, -10 minutes and Choose a time. With a controlled action time of 17:00, the shortcuts declare 17:00, 16:55 and 16:50 respectively; manual selection uses the chosen time. Recording time remains distinct and existing timing validation applies.
- One action records the declared 17:00 exit and preserves the original planning basis. Show 15 estimated minutes with a pattern-derived 16:45 start and an explicit pending synchronization status. No arrival check-in is required.
- Close and restart the app offline within the access period. The fact, actor, operation identity and planning basis survive. B has no invented indication of completion before receiving the fact.
- Restore connectivity. After server acceptance and synchronization, both phones show the same fact and provenance, with no pending status for the acknowledged operation.
- Lose the acknowledgement after the server commits, then retry. Exactly one business fact and one accepted operation result remain. A changed payload reusing the same operation identity is rejected.
- If B independently declares another exit, retain both facts as a suspected duplicate, exclude the affected duration and mark the total incomplete until explicit resolution. Today exposes an À vérifier entry point. Opening it shows both declarations and supports an explicit retained-fact choice with history preserved; no automatic winner is preselected.
- A changed plan while A was offline cannot silently replace the original basis. Incompatible reconciliation requires review. Temporary failures retain pending work; durable business rejection remains visible for action while independent valid work can progress.
- Switching accounts cannot upload A's work as B. An expired grant pauses upload; confirmed membership removal prevents replay under a later invitation.

Pass evidence: two physical iPhones, offline restart, network restoration, lost acknowledgement, duplicate conflict and account isolation. Existing simulator evidence is only a precursor.

## J05: Correct 17:48 to 17:43 with history

Given a care exit declared at 17:48 and a pattern-derived start at 16:30:

- A opens the existing declaration, chooses 17:43 and supplies a correction reason. Append a correction retaining the original 17:48 declaration, author, reason and recording history.
- The current reading becomes 73 estimated minutes instead of 78. The pattern-derived start remains identified; editing the end does not turn it into a declared arrival.
- If an arrival was separately declared at 16:40, the same end correction instead changes 68 declared minutes to 63. Show the original and corrected readings in history.
- An offline correction is shown as pending, survives restart and converges without duplicate corrections after retry. B can inspect the accepted history after synchronization.
- Competing corrections, or a correction competing with annulment, remain unresolved regardless of arrival order. Exclude the affected duration until explicit resolution, retaining all facts.
- Immediately after a declaration, Undo appends an annulment and preserves the original Punch in history. Exercise it before upload, across restart and after acceptance; replay must preserve the original and its annulment without duplication. Normal conflict rules still apply if another adult corrected the same fact.
- Annulment remains a separate traceable action. An invalid end before the effective start is not included as a negative duration or silently repaired; show Needs completion.

Pass evidence: both provenance variants, the history on both phones, an offline retry and both arrival orders of a competing correction set.

## J06: Compare a month with missing data visible

Given a selected household-local calendar month and person/activity filters:

- The monthly comparison is available in the application and as an Excel (.xlsx) export matching the selected month and person/activity filters. Verify French column labels, separate actual arrival/departure and retained start/end, separate elapsed and counted durations/totals, applied-rule context, status, generation time, timezone and synchronization freshness.
- Export the same month as separate files per child and as one combined workbook. Verify one École worksheet per child and one separate worksheet per child/caregiver pair. Two children with one caregiver each produce four sheets; a child with two caregivers has three sheets including École. École includes care and school lunch; Assistante maternelle includes daily planned/actual/retained times, normal and supplementary hours, meals, snacks, subtotals and monthly totals. Each separate child file contains only that child's school and caregiver sheets with readings identical to the combined workbook. Test duplicate first names and title collisions without mixing identities. Keep units and actual/retained totals separate; food and hour classification rules must be specified before acceptance. No other child's data appears in an individual file.
- For a child with two concurrent caregivers, separate their daily rows and subtotals by arrangement, applying each contract independently. Do not mix caregiver-supplied meal/snack counts or hide the provider behind a combined child total. Verify separate child/caregiver worksheets, identifiable names, independent totals and no leakage from another caregiver into those totals.
- For actual 08:47-17:25 and retained 09:00-17:30 under the 30-minute example, verify 518 actual minutes and 510 counted minutes in both app and workbook. Missing/conflicting readings must remain identifiable without fabricated zeros or inclusion in ineligible totals. Open the workbook in Excel and compare with the same application snapshot; specification alone is not export acceptance.
- Count only caregiver-supplied meals and snacks in the nanny recap. A day with a parent-supplied meal and a caregiver-supplied snack contributes zero caregiver meals and one caregiver snack, subject to the confirmed absence and correction rules. App and Excel use identical meanings and totals.
- Configure meals and snacks in the nanny weekly schedule or typical-day definition, including days with neither, meals only, snacks only or both. Preserve selected-day independence and dated revisions. Verify planned quantities without inventing consumption from care duration; count planned food under the confirmed rule, exclude items affected by child/caregiver absence and support traceable per-occurrence corrections. When planning the dated change in advance or recording collection/a half-day change, provide independent Remove meal and Remove snack choices. Verify either, both and neither, with no required food times and no automatic deduction based on collection time. Preserve other days and the typical-day plan. Advance exclusions survive synchronization and later collection without reset or double deduction; explicit later changes remain traceable. Future items remain planned and inferred quantities remain labelled as such. Any compensation between normal and supplementary hours remains pending confirmation.
- Display planned quantities separately from observed readings. Distinguish declared and presumed meals, declared durations and durations with a pattern-derived boundary. Identify sessions excluded for missing information or conflict.
- Declare a child's whole-day or half-day absence with explicit boundaries and a confirmed activity list. Fully covered tracked sessions receive absence declarations; lunch presumption is overridden. A planning-only cancellation is labelled separately, partial overlaps require explicit adjustment, existing contradictory attendance stays in conflict, and other children and the recurring template remain unchanged. No medical reason is required.
- For a nanny/nursery absence, prefill retained hours from the applicable contract and allow case-by-case confirmation or change. Exercise zero and nonzero retained hours without requesting a reason, proof status or attachment. Both remain declared absences with no fabricated actual attendance; the app and Excel distinguish retained absence hours from actual care duration. Preserve adjustment history.
- A planned lunch remains excluded from taken meals before its slot ends. Afterwards, verified calendar coverage permits presumption unless explicit absence or conflict overrides it. Annulled absence restores presumption only when those conditions still hold.
- No morning entry or afternoon exit means normal school arrival/pick-up is assumed, with no recorded care and no incomplete-care warning. It is not verified attendance. Unknown data never silently becomes a measured zero.
- Caregiver setup offers Exact minutes or Increments of 5, 10, 15 or 30 minutes. Exercise every option on its wall-clock grid and preserve the applied rule version. With arrival 08:47 and departure 17:25 under the example tolerance windows, Exact minutes counts 09:00-17:25 (505 minutes); the 30-minute option counts 09:00-17:30 (510 minutes). Changing the setting prospectively does not rewrite earlier counted durations.
- For the confirmed example contract with baseline 09:00-17:00 and windows 08:45-09:00 and 16:45-17:00, declarations at 08:52 and 16:53 retain 481 elapsed minutes in history but contribute 480 contract-counted minutes. Declarations at 08:47 and 17:25 retain the original times and produce retained boundaries 09:00-17:30, or 510 counted minutes, under the example contract rule. Explain the applied version and distinguish both duration types. Do not generalize this rounding rule to other contracts or fabricate missing declarations.
- For the same nanny contract with 30-minute rounding, arrival at 09:10 and departure at 16:30 count as 09:00-16:30, or 450 minutes. Departure at 16:20 also retains 16:30. Preserve both raw declarations; do not enforce an unconditional eight-hour minimum. Test the separately configured 15-minute increment without silently reusing the 30-minute rule.
- With contractual start 09:00, arrival tolerance 08:45-09:00 and a 30-minute increment, arrival at 08:35 retains a counted start of 08:30. Arrival at 08:47 still retains 09:00; arrival at 08:10 retains 08:00, applying two increments rather than capping at one. Preserve the raw arrival in every case and show the applied contract rule.
- For the corrected 15-minute wall-clock grid, outside the configured tolerance, arrival at 08:35 rounds to 08:30 and departure at 17:10 rounds to 17:15. Exact quarter-hour boundaries remain unchanged. A non-round contractual start does not shift the grid. Keep 30-minute examples scoped to that configured contract option.
- In nanny/nursery and school-care settings, Increments exposes the 5/10/15/30-minute step and Up/Down direction; Exact minutes applies neither. Persist both with the policy version, preserve historical readings and display the applied rule. Round clock-time boundaries by default, without a duration-versus-time selector. Test both directions, exact boundaries and one-minute deviations; previous examples apply only to their explicit policies. Arrival defaults to Down and departure to Up. Verify that each can be changed independently for the applicable arrangement and that changing one leaves the other unchanged.
- School-care setup offers Exact minutes or 5/10/15/30-minute increments. Preserve the selected version and show source duration and retained counted duration separately in the app and École worksheet, including pattern-derived provenance. Round applicable clock-time boundaries before deriving counted duration; do not round the elapsed duration itself or copy nanny tolerance/baseline rules implicitly.
- For morning entry at 07:45 and planned school start at 08:30, show 45 expected minutes before 08:30, excluded from the calculated total. At 08:30 include 45 estimated minutes. An explicit 08:10 exit instead contributes 25 declared minutes.
- Suspected duplicates, contradictory attendance and unresolved corrections do not contribute to their affected aggregate. Show the excluded session count and mark that aggregate incomplete. A valid estimated duration is not excluded merely for using the pattern.
- Keep offline/pending freshness visible independently of missing business data. Late accepted declarations and explicit corrections update the current month reading with traceable history; future plan edits do not rewrite its historical basis.
- Both phones with the same complete fact set produce identical readings and totals, including when facts arrive in reverse order. Filters and month boundaries use the household timezone.

Minimum numeric fixture: three ended, calendar-verified planned lunches comprising one declared presence, one presumed meal and one declared absence yield three planned meals and two taken meals, split as one declared and one presumed. Four care occurrences comprising 73 estimated minutes, 63 declared minutes, one unresolved duration and one with no care declaration yield 136 calculated minutes, split 73/63, with one excluded session and an incomplete duration total. The undeclared care occurrence adds neither minutes nor an incomplete session.

Pass evidence: inspect the category breakdown, excluded-session drill-down, month boundaries, correction history and order-independent convergence. A bare combined total is insufficient.

## Additional V1 support requirement

The product owner added a support, questions and feature-request form after confirming the original six journeys. Its scope and S01-S11 checks are specified in [support and feedback](19-support-and-feedback.md). These checks supplement the six journeys and are also required for V1 acceptance.

## Acceptance and evidence ledger

| Scope | Available evidence | Required before acceptance |
| --- | --- | --- |
| J01-J03 planning | Selected in-memory examples in the domain prototype | Complete application journeys, shared-account behavior and concurrency |
| J04 offline exchange | Two-simulator native slice, durable queue and replay experiments | Physical devices, lifecycle/network cases, production authorization and full conflict behavior |
| J05 correction | Domain examples and a minimal native correction conflict | Complete history interface, competing resolution behavior and both delivery orders |
| J06 monthly comparison | Selected domain calculations | Full reducer, monthly interface, aggregate fixtures and convergence |
| N01-N10 notifications | Specification only | Selected provider integration and physical-device execution |
| Access and privacy | Policy plus limited simulator revocation experiments | Independent revalidation, encryption, restart-safe purge and the full policy matrix |

See the [domain report](13-prototype-verification.md), [native report](16-native-sync-prototype.md) and [access acceptance matrix](17-access-and-privacy.md). Record actual results and unresolved failures against criterion IDs when implementation exists. Specification completion does not close physical-device, backend failure, erasure, recovery, retention or deployment gates.
