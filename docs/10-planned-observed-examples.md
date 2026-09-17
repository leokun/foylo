# Planned and observed: confirmed examples

The current prototype follows the confirmed one-action interaction. With no declaration, normal school arrival or pick-up is assumed and no care is counted. Morning Care entry supplies the declared start; afternoon Care exit supplies the declared end. The pattern supplies the other boundary. Earlier automatic-care-activation and missing-exit interpretations are superseded.

These examples record the product owner's confirmations in [Planned/observed rules](https://github.com/leokun/foylo/issues/7). They describe single-day sessions with an unambiguous applicable plan, including the transition at the end of a lunch slot. Historical plan versioning, conflicting calendars and concurrent corrections are now specified in the [V1 business decisions](12-v1-business-decisions.md), with a separate validation matrix. A later [verification pass](13-prototype-verification.md) exercises selected defaults with explicit in-memory fixtures; its limits remain separate from the earlier results below.

| Situation | Confirmed interpretation |
| --- | --- |
| School begins at 08:30; Care entry declared at 07:45 | Automatically record 07:45-08:30, giving 45 minutes with a declared start and pattern-derived end. |
| At 08:00, entry declared at 07:45 and school starts at 08:30 | Care in progress; show entry 07:45, planned end 08:30 and expected duration 45 minutes. Exclude the 45 minutes from the calculated total until 08:30. |
| At 08:30, the same valid morning interval reaches its pattern-derived end | Include 45 minutes in the calculated total, keeping the pattern-derived end visible. No manual exit is required. |
| Entry declared at 07:45, then an exceptional morning exit declared at 08:10, with school planned at 08:30 | Replace the pattern-derived end with the declared exit. Include 25 declared minutes in the total, with history preserved. |
| Morning-care pattern with no entry declaration | Normal school arrival assumed; no care recorded and no incomplete-care warning. |
| Lunch before the planned slot ends, with no declaration | Planned only, excluded from the taken-meal total. |
| Lunch at or after the planned slot end, with no declaration | One presumed meal, distinct from declared attendance. |
| Declare lunch absence | The presumed meal is replaced by declared absence. |
| Annul that absence, with the lunch still planned | The meal becomes presumed again after the planned slot ends, or planned before then; the absence and its annulment remain in history. |
| Afternoon care planned from 16:30; Care exit declared at 17:48 | 78 minutes with an estimated start. |
| Subsequently declare arrival at 16:40 | 68 minutes based on declared endpoints, with history retained. |
| Annul that arrival | 78 minutes with the planned 16:30 start as an estimate again. |
| Correct a single pick-up from 17:48 to 17:43 | The correction replaces the earlier value in the current reading and preserves it in history. With an estimated 16:30 start, the duration is 73 minutes. |
| Two independently declared pick-ups for the same child and session | Flag a suspected duplicate, choose neither automatically, exclude the affected duration and mark the total incomplete. |
| Explicitly keep the correct pick-up and annul the other | Recalculate from the retained declaration, preserving both declarations and the annulment in history. |
| Scheduled care slot with no exit declaration | Normal school pick-up assumed; no care recorded and no incomplete-care warning. |
| Explicit care exit with no usable start | Mark Needs completion; exclude the duration and mark the total incomplete. A usable pattern start is sufficient. |
| Explicit Care exit before the usable care start on a single-day session | Mark Needs completion instead of contributing an inconsistent duration. |

The monthly summary separates presumed and declared meals, estimated and fully declared durations, and shows the number of sessions that need completion. Unknown duration must not silently appear as zero.

## Latest confirmations: morning interval in progress and explicit exit

The product owner confirmed the live display, total transition and optional explicit morning exit described above. These are confirmed requirements; the later verification report distinguishes executed examples from pending browser checks. The earlier verification below predates these clarifications. The later [verification pass](13-prototype-verification.md) executes the 08:00, 08:29 and 08:30 aggregation boundaries, the exceptional 08:10 exit, and its annulment. Browser display and interaction remain unverified.

## Prototype boundary

The throwaway demonstration is local and in memory. It does not implement authentication, persistence, synchronization, production occurrence identity or time-zone handling. Simple schedule/calendar/exception fixtures exercise on-demand expansion without adopting a calendar precedence rule. Ambiguous calendar inputs and concurrent corrections must be exposed rather than silently resolved.

## Verification on September 17, 2026

The updated local TypeScript prototype was executed with in-memory examples. Observed results:

- Morning Care entry at 07:45 with school starting at 08:30: 45 minutes, declared start and pattern-derived end.
- Afternoon Care exit at 17:00 with care starting at 16:45: 15 minutes, pattern-derived start and declared end.
- No declarations for either care pattern: two normal-school defaults, no care duration and zero incomplete sessions.
- Annulment of the sole morning entry or afternoon exit: the corresponding normal-school default is restored with history retained.
- Independent duplicate morning entries or afternoon exits: Needs completion, with the affected duration excluded until explicit resolution.
- Correcting a morning entry from 07:45 to 07:50 gives 40 minutes. Correcting an afternoon exit from 17:00 to 17:10 gives 25 minutes.
- Two explicitly declared endpoints produce declared-duration provenance. A malformed explicit time remains Needs completion.
- Lunch ending at 13:00: planned at 12:59, presumed at 13:00 unless declared otherwise.
- A combined example with one morning entry, one afternoon exit and one elapsed planned lunch: 60 minutes with a pattern-derived boundary, one presumed meal, and a complete summary.

The rebuilt self-contained HTML embeds the updated domain module and includes guided examples for morning care, afternoon care, both duplicate types, a correction and the lunch boundary. Its JavaScript syntax was checked. Browser verification remains incomplete after the earlier browser-control timeouts; no interactive acceptance is claimed.

Previous examples of append-only correction, annulment, unresolved concurrent corrections and unsupported calendar precedence remain part of the investigation. The prototype still does not choose a winner for concurrent facts or a priority for conflicting calendars.

Local artifacts for this session: `/tmp/foylo-planned-observed-domain.ts`, `/tmp/foylo-planned-observed-domain.js` and `/tmp/foylo-planned-observed-prototype.html`. They are temporary files, not committed or published deliverables. This note preserves the verified examples and limitations in the design dossier.

The meal transition is confirmed at the planned slot end. The prototype uses an explicit local clock for this boundary; the adopted time-zone, daylight-saving and offline-clock rules are specified in the [V1 business decisions](12-v1-business-decisions.md#time-and-offline-interpretation) and remain unverified in the prototype.
