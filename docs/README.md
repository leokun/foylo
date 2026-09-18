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
17. [Decision register](adr/README.md)

## Statuses

- **Decided**: choice retained in the final synthesis or explicitly confirmed since.
- **To validate**: proposal, design detail or trade-off still open.
- **Rejected/deferred**: option abandoned or postponed, with its reason.

Examples and validation criteria refine the plan; they do not constitute additional validation of the scope.

The product owner has confirmed daily coordination as the primary value, the [V1 scope](01-v1-scope.md), and its six essential validation scenarios. The remaining planning and planned/observed rules now have adopted V1 defaults. Full journey acceptance criteria remain to be completed.

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

1. Use the [executed native prototype](16-native-sync-prototype.md) to define access revalidation, offline lifetime and revoked-cache cleanup, then close the remaining physical-device, recovery, full conflict-reducer and backend failure gates. SQLite/PowerSync remains the preferred candidate; no production engine or hosting choice is finalized.
2. Complete access/privacy/notification requirements and the detailed V1 journey acceptance criteria, including technical constraints that affect those choices.
3. Assemble the V1 specification and implementation sequence.
4. Retain browser verification and the unmodeled integration cases as pending acceptance work; the local-file browser check was blocked by browser security policy. The [20 executed prototype scenarios](13-prototype-verification.md) remain limited evidence.

Continue with reasonable V1 defaults under the delegated mandate. Revisit a decision if implementation evidence exposes a material constraint.

Earlier confirmed rules and their limited prototype results are recorded in the [worked examples](10-planned-observed-examples.md). They now cover the pattern-default and morning-entry/afternoon-exit interactions. Browser verification of the demonstration remains incomplete and is not production acceptance.

The isolated backend and first native synchronization experiments have executed evidence with explicit limits. The native slice works on two simulators, including bundled offline restart, idempotent retry and account queue isolation. Revocation revalidation and broader acceptance remain open. Starting the production application remains a separate step.
