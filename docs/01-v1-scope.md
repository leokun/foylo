# V1 scope

## Decided

The V1 and the data schema must be framed before any coding starts. The offline operation principle is retained.

## To validate: proposed scope

| Domain | V1 proposal |
| --- | --- |
| Household | One family per account in V1 usage, two adult accounts |
| Persons | Children and adults represented independently of accounts |
| Reference data | Places and activities |
| Planning | Weekly template, school calendars, public holidays, closures and exceptions |
| Responsibilities | Who drops off or picks up each person |
| Today | Activities, times, places, responsible adults and next action |
| Check-in | Start/end, end only, present/absent, quick adjustment |
| History | Traceable corrections and planned/observed separation |
| Summary | Monthly quantities and durations per person and activity |

The quick actions under consideration are "Now", "-5 minutes", "-10 minutes" and picking a time. For after-school care starting at 16:30, a pick-up at 17:48 can be entered on its own. The provenance of the start time used in the calculation must remain explicit.

School lunch must be plannable automatically without requiring a daily confirmation. How meals without a check-in are qualified remains to be decided: expected or presumed, but not presented indiscriminately as observed attendance.

## Proposed scenarios to validate the V1

1. Set up two children with different places and weekly templates.
2. Automatically exclude periods when the school is closed.
3. Change the responsible adult on one Thursday without affecting the following Thursdays.
4. Record a pick-up offline, then find the fact on the second phone.
5. Correct 17:48 to 17:43 while keeping the history of the correction.
6. Compare planned and declared over a month, with missing data identifiable.

## Rejected/deferred

Explicit rounds, navigation, pricing, Android, web, Watch, multiple households, sophisticated external permissions and rich push are proposed for after the V1. See [future scope](09-future-scope.md).
