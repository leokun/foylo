# V1 scope

## Decided

The V1 and the data schema must be framed before any coding starts. The offline operation principle is retained.

## Confirmed V1 scope

The product owner confirmed this scope and the six validation scenarios on September 17, 2026 in [Confirm the V1 scope table and the six validation scenarios](https://github.com/leokun/foylo/issues/3). Delivery is iPhone-only, for one household shared by two adult accounts. Detailed business rules and acceptance criteria remain to be defined.

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
| Summary | Monthly quantities and durations per person and activity |

## Schedule pattern and one-action care recording

The latest product clarification supersedes the earlier interpretation of care automatically activating because no pick-up was recorded.

The example pattern defines school ending at 16:30 and a possible after-school-care start at 16:45. With no declaration, the normal assumption is collection at school dismissal: no care is recorded and no care duration is marked missing. This assumption is not a verified pick-up or an automatically authored declaration.

If the parent instead selects **Care exit** at 17:00, the application automatically records the interval's start at 16:45 from the pattern and its end at 17:00 from that action: 15 minutes of care. No arrival check-in, manual start or manual cancellation is required. Both boundaries are available in the recorded result, with their provenance distinguished: pattern-derived start and parent-declared end. Their representation as events or interval data is not yet chosen.

For morning care, the complementary rule is confirmed: if school starts at 08:30, selecting **Care entry** at 07:45 automatically records a 07:45-08:30 interval, giving 45 minutes. The start is parent-declared and the end is supplied by the school-start time in the pattern. No care-exit declaration is required for that interval. With no entry declaration, normal arrival at school is assumed, with no morning care recorded and no incomplete-care warning. The pattern-derived end is not an independently observed exit; handling the live interval before 08:30 remains a separate display question.

Passing 16:45 without any action does not itself establish care attendance. The exit declaration establishes that care took place and provides the information needed to reconstruct its interval. This default is specific to the school/care pattern; it does not replace the separately confirmed lunch presumption.

Earlier proposals for mandatory generic child-pick-up declarations, automatic care activation in their absence, cross-day cancellation/recreation and future-session deletion workflows are not accepted. Dated-version storage also remains undecided. Offline reconciliation, corrections and the meaning of other patterns remain to be specified.

## Confirmed planned/observed rules

The product owner confirmed the following rules on September 17, 2026 in [Planned/observed rules](https://github.com/leokun/foylo/issues/7):

- A planned school lunch without a check-in becomes presumed taken at the end of its planned time slot, unless an explicit declaration says otherwise. Before that boundary it remains planned and is excluded from the taken-meal total. No daily confirmation is required, and presumed meals remain distinguishable from explicitly declared attendance.
- When the parent declares an afternoon care exit, calculate the duration from its planned start and clearly identify that start as estimated. For a planned start at 16:30 and a declared pick-up at 17:48, the resulting 78 minutes use an estimated start, not a declared arrival.
- For morning care, a declared entry combines with the school-start time from the pattern to record both boundaries automatically; the end retains its pattern-derived provenance.
- An explicitly declared school-lunch absence replaces the presumption that the meal was taken. Conflicting explicit declarations still require a separate resolution rule.
- An arrival time declared afterwards replaces the estimated start in the duration calculation. The history is preserved. For an arrival later declared at 16:40 and a pick-up at 17:48, the duration becomes 68 minutes based on declared endpoints.
- For an explicitly declared care event, a missing required time or inconsistent timing is marked **Needs completion**. Its duration is excluded from the calculated total, and that total is explicitly marked incomplete. The confirmed planned-start fallback still applies when only the actual arrival declaration is missing: a valid estimated duration is not excluded merely because one boundary comes from the pattern.
- A check-in entered by mistake can be annulled while retaining its history. Annulment is not a silent deletion.
- Two check-ins of the same type for the same child and the same session are flagged as a suspected duplicate. Neither is automatically chosen as the one to retain. This is distinct from replaying the same technical operation.
- The monthly summary distinguishes declared and presumed meals, and durations based on declared endpoints from durations using a pattern-derived start or end. It also shows the number of sessions that still need completion.

- Resolve a suspected duplicate by explicitly choosing the correct check-in and annulling the other, retaining both histories. Until resolved, exclude the affected duration from the total and identify the summary as incomplete.
- After annulment, recalculate using the remaining information. For afternoon care, annulling a separately declared arrival restores the pattern-derived start while a care exit remains. Annulling the only morning care entry or afternoon care exit leaves no care declaration and restores the normal school arrival or pick-up assumption. Annulment of a lunch absence restores the presumed meal once the planned slot has ended, when the planned lunch remains and no other declaration overrides it. Before that boundary it returns to planned. A real absence must therefore remain explicitly declared.

A scheduled morning-care slot with no entry declaration follows the normal-school-arrival assumption; a scheduled afternoon-care slot with no exit declaration follows the normal-school-pick-up assumption. Neither is an incomplete care record. These rules do not create a declared presence or arrival from planning alone. Missing duration must not silently become zero. Concurrent corrections, competing duplicate resolutions and contradictory explicit attendance declarations remain separate open cases.

## Confirmed planning-history principle

Weekly-template changes take effect from a chosen date without modifying earlier sessions or their calculations. A historical correction must be an explicit, traceable action. School-calendar updates also preserve past sessions and calculations; any retroactive correction requires an explicit, traceable action. The representation of planning versions remains open in [Occurrence identity and preservation of the past](https://github.com/leokun/foylo/issues/8).

Two time slots on the same day are distinct sessions, including when they concern the same activity. A one-off addition also has its own identity. Changing the time of a session without changing its day preserves its identity and the attachment of its check-ins and exceptions. This does not authorize silent recalculation of historical sessions.

## Details still to validate

The quick actions under consideration are "Now", "-5 minutes", "-10 minutes" and picking a time. Concurrent corrections, competing duplicate resolutions and contradictory explicit attendance declarations remain open in [Concurrent facts](https://github.com/leokun/foylo/issues/10). Historical plan changes and calendar precedence remain in their dedicated decisions.

## Confirmed V1 validation scenarios

These are required journeys, not completed acceptance tests. Their detailed rules and acceptance criteria remain open.

1. Set up two children with different places and weekly templates.
2. Automatically exclude periods when the school is closed.
3. Change the responsible adult on one Thursday without affecting the following Thursdays.
4. Record a pick-up offline, then find the fact on the second phone.
5. Correct 17:48 to 17:43 while keeping the history of the correction.
6. Compare planned and declared over a month, with missing data identifiable.

## Rejected/deferred

Explicit rounds, navigation, pricing, Android, web, Watch, multiple households, sophisticated external permissions and rich push remain outside V1. See [future scope](09-future-scope.md). Deferral does not commit any later delivery.
