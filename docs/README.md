# Foylo: design dossier

Updated September 18, 2026. Current phase: documentation and decisions, no implementation.

These documents draw on the "Family check-in applications" conversation of September 17, 2026, in particular its full brief, its counter-analysis and its final synthesis. Earlier proposals are kept as leads where they have not been confirmed. The initial transcription did not include new technical, commercial or legal research. Subsequent research is listed separately below and does not itself confirm a product or architecture choice.

## Reading order

1. [Product vision](00-product-vision.md)
2. [V1 scope](01-v1-scope.md)
3. [Domain model](02-domain-model.md)
4. [Architecture](03-architecture.md)
5. [Offline operation and synchronization](04-offline-sync.md)
6. [Authentication and permissions](05-auth-permissions.md)
7. [Notifications](06-notifications.md)
8. [Platforms](07-platforms.md)
9. [Security and privacy](08-security-privacy.md)
10. [Future scope](09-future-scope.md)
11. [Planned and observed examples](10-planned-observed-examples.md)
12. [V1 business decisions](12-v1-business-decisions.md)
13. [Prototype verification](13-prototype-verification.md)
14. [Backend compatibility prototype](14-backend-compatibility-prototype.md)
15. [Mobile synchronization baseline and validation](15-mobile-sync-validation.md)
16. [Native synchronization prototype](16-native-sync-prototype.md)
17. [V1 access and local privacy policy](17-access-and-privacy.md)
18. [V1 journey acceptance criteria](18-v1-journey-acceptance.md)
19. [V1 support and feedback](19-support-and-feedback.md)
20. [V1 specification and implementation sequence](20-v1-specification-and-delivery-plan.md)
21. [Lifecycle, erasure and recovery](21-lifecycle-erasure-and-recovery.md)
22. [Decision register](adr/README.md)

## Statuses

- **Decided**: choice retained in the final synthesis or explicitly confirmed since.
- **To validate**: proposal, design detail or trade-off still open.
- **Rejected/deferred**: option abandoned or postponed, with its reason.

Examples and validation criteria refine the plan; they do not constitute additional validation of the scope.

The product owner has confirmed daily coordination as the primary value, the [V1 scope](01-v1-scope.md), and its six essential validation scenarios. The remaining planning and planned/observed rules now have adopted V1 defaults. France-only school calendars and the simple one-off activity form are confirmed in the scope. The [six journey acceptance criteria](18-v1-journey-acceptance.md) and [notification policy](06-notifications.md) are specified; implementation and end-to-end acceptance remain pending.

## V1 specification map and research

The [V1 specification map](https://github.com/leokun/foylo/issues/1) tracks decisions and their dependencies. Its agreed destination covers user journeys with acceptance criteria, planned/observed business rules, the conceptual data model, and the technical foundation. SQL schema, API payloads, screen designs and repository tooling are outside this map.

Research findings checked on September 17-18, 2026:

- [French school calendars and institution-specific closures](research/school-calendars.md).
- [Offline synchronization: PowerSync, ElectricSQL and a custom protocol](research/offline-sync.md).
- [Mobile storage and synchronization with the selected backend](research/mobile-storage-and-sync.md).
- [Backend and authentication: Supabase and a dedicated API](research/backend-and-auth.md).
- [Better Auth: PostgreSQL, Expo and local validation](research/better-auth.md).
- [Effect and database compatibility, including MikroORM and Prisma](research/effect-database.md).

These notes provide evidence and candidate approaches for the open decisions. Recommendations, unresolved questions and validation work are identified within each note.

## Current clarification

The school/care pattern assumes normal school pick-up when no action is entered. Selecting Care exit at 17:00 records the start from the pattern (16:45) and the declared end (17:00), giving 15 minutes. The morning counterpart is confirmed: Care entry at 07:45 plus an 08:30 school start records 45 minutes with a pattern-derived end. Without an entry, normal school arrival is assumed. A clock threshold alone does not establish care attendance. This supersedes the earlier generic-pick-up/automatic-activation interpretation. The local domain prototype now models both interactions and its examples have been exercised. Visual verification remains incomplete; the planned/observed investigation remains open for review.

The morning live display is now confirmed: before the school-start boundary, show Care in progress, the declared entry, the planned end and the expected duration. The duration enters the calculated total at that boundary while retaining its pattern-derived provenance. An optional morning exit declared at 08:10 after a 07:45 entry replaces the planned 08:30 end and gives 25 declared minutes, preserving history. These morning rules now have executed in-memory examples in the [verification report](13-prototype-verification.md); browser interaction remains unverified.

Future exceptions already entered are preserved after a weekly-template change. Any exception made incompatible by the new plan is flagged for review, without automatic deletion. If its slot is removed, the session stays visible as To resolve until explicitly kept as a one-off activity or cancelled, with history preserved.

A one-off activity moved to another day before any check-in is recorded keeps its identity, information and responsible adult, with a traceable date change. An appointment that actually took place stays in history; another appointment is a new occurrence. A mistaken attendance declaration must be explicitly annulled before moving the same appointment, with history retained. Recurring-session moves and remaining business cases now follow the adopted [V1 business decisions](12-v1-business-decisions.md).

The [conceptual data model](02-domain-model.md#frozen-concepts-and-responsibilities) is now frozen for V1: household relationships, stable occurrence references, planning revisions, Punch actions and derived readings. [ADR 0005](adr/0005-occurrence-references-and-planning-basis.md) records the structural choice. This is a documentation baseline, not a database implementation.

The backend stack is now selected: NestJS, Effect, Prisma 7, PostgreSQL and Better Auth, with separate API/core/data packages. [ADR 0007](adr/0007-backend-stack-and-package-boundaries.md) supersedes server-side Drizzle. The [isolated prototype](14-backend-compatibility-prototype.md) passed 11 local checks. Production interruption semantics, dependency advisories, hosting, mobile storage and synchronization remain open.

## Next work, in order

The product owner delegated the remaining business choices instead of continuing per-case confirmation. The [V1 business decisions](12-v1-business-decisions.md) now settle occurrence identity, planning revisions, calendar precedence, concurrent facts and time interpretation. They supplement earlier confirmations and supersede earlier open-status wording on those subjects.

The [consolidated V1 specification and implementation sequence](20-v1-specification-and-delivery-plan.md) maps each requirement to its detailed source, acceptance criteria and future implementation lot. It includes the support form and keeps all release gates explicit.

The monthly recap now includes Excel export, superseding the earlier in-app-only scope. Recorded and contract-retained times and durations remain distinct in both views. Export offers separate files per child or one combined workbook, with one [First name] École worksheet per child and one separate worksheet per child/caregiver pair. These cover school care/lunch and nanny times, normal/supplementary hours, meals, snacks, subtotals and monthly totals. Nanny meals/snacks are planned in the weekly or typical-day setup and count only food supplied by the caregiver, excluding parent-supplied food. Food counts follow the plan, exclude items affected by child/caregiver absence and permit corrections. In advance when editing the dated day, or at collection/a half-day change, the parent explicitly chooses whether to remove the meal, snack or both; no food-time configuration is required. Possible compensation between normal and supplementary hours is explicitly unconfirmed. Nanny leave days must be recorded separately from child absences and training, including in Excel; their contract-hour treatment remains to confirm, with no assumed zero-hour default.

School care and nanny/nursery accounting offer exact minutes or 5/10/15/30-minute increments with configurable Up/Down rounding. Clock-time boundary rounding is selected by default, with no basis selector. Arrival and departure directions are independently configurable, defaulting to Down and Up respectively; source times and pattern-derived provenance remain preserved.

A child can have multiple concurrent caregivers with independent days, calendars, contracts and food rules, including one caregiver on Monday/Tuesday/Thursday/Friday and another on Wednesday. Reconcile these arrangements with the conceptual model; Excel presentation is confirmed as one sheet per child/caregiver pair.

The confirmed [caregiver contract windows and counted duration](01-v1-scope.md#confirmed-caregiver-contract-windows-and-counted-duration) distinguish exact declarations from contract-retained time. Exact-minute or 5/10/15/30-minute increment accounting is selected. Absence hours are confirmed case by case from a contract default, without justification tracking. Reconcile the conceptual model and monthly summary before freezing persistence contracts.

1. Validate automatic same-verified-email account matching for the selected [Apple, Google and email sign-in links](05-auth-permissions.md), including Apple relay addresses. Backup email and manual recovery are deferred. The [lifecycle policy](21-lifecycle-erasure-and-recovery.md) now specifies erasure, closure, retention and the no-override recovery boundary; provider feasibility and policy review remain release gates.
2. Validate the adopted [access policy](17-access-and-privacy.md) and remaining physical-device, encrypted-storage, recovery, full conflict-reducer and backend failure gates. SQLite/PowerSync remains conditional.
3. Once production implementation starts, follow the dependency-ordered lots in the consolidated plan. Support requests will stay in the Foylo database with an internal feature-prioritization list; validate reviewer access and retention for that lot. Select a notification provider before notification integration.
4. Execute the [six journeys](18-v1-journey-acceptance.md), [notification checks](06-notifications.md) and [support checks](19-support-and-feedback.md) against the assembled product. Retain browser and unmodeled integration verification as pending; existing prototype results remain limited evidence.

Continue with reasonable V1 defaults under the delegated mandate. Revisit a decision if implementation evidence exposes a material constraint.

Earlier confirmed rules and their limited prototype results are recorded in the [worked examples](10-planned-observed-examples.md). They now cover the pattern-default and morning-entry/afternoon-exit interactions. Browser verification of the demonstration remains incomplete and is not production acceptance.

The isolated backend and first native synchronization experiments have executed evidence with explicit limits. The native slice works on two simulators, including bundled offline restart, idempotent retry and account queue isolation. Revocation revalidation and broader acceptance remain open. Starting the production application remains a separate step.
