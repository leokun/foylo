# Foylo: design dossier

Updated September 17, 2026. Current phase: documentation and decisions, no implementation.

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
12. [Decision register](adr/README.md)

## Statuses

- **Decided**: choice retained in the final synthesis or explicitly confirmed since.
- **To validate**: proposal, design detail or trade-off still open.
- **Rejected/deferred**: option abandoned or postponed, with its reason.

Examples and validation criteria refine the plan; they do not constitute additional validation of the scope.

The product owner has confirmed daily coordination as the primary value, the [V1 scope](01-v1-scope.md), and its six essential validation scenarios. Detailed business rules and acceptance criteria remain open.

## V1 specification map and research

The [V1 specification map](https://github.com/leokun/foylo/issues/1) tracks decisions and their dependencies. Its agreed destination covers user journeys with acceptance criteria, planned/observed business rules, the conceptual data model, and the technical foundation. SQL schema, API payloads, screen designs and repository tooling are outside this map.

Research findings checked on September 17, 2026:

- [French school calendars and institution-specific closures](research/school-calendars.md).
- [Offline synchronization: PowerSync, ElectricSQL and a custom protocol](research/offline-sync.md).
- [Backend and authentication: Supabase and a dedicated API](research/backend-and-auth.md).

These notes provide evidence and candidate approaches for the open decisions. Recommendations, unresolved questions and validation work are identified within each note.

## Current clarification

The school/care pattern assumes normal school pick-up when no action is entered. Selecting Care exit at 17:00 records the start from the pattern (16:45) and the declared end (17:00), giving 15 minutes. The morning counterpart is confirmed: Care entry at 07:45 plus an 08:30 school start records 45 minutes with a pattern-derived end. Without an entry, normal school arrival is assumed. A clock threshold alone does not establish care attendance. This supersedes the earlier generic-pick-up/automatic-activation interpretation. The local domain prototype now models both interactions and its examples have been exercised. Visual verification remains incomplete; the planned/observed investigation remains open for review.

## Next decisions, in order

1. Review the updated one-action care prototype, then revisit occurrence identity and planning versions. Distinct daily slots, one-off identities, identity preservation for same-day time changes, and protection of the past under weekly-template and school-calendar changes are confirmed.
2. Set calendar/exception precedence and resolve concurrent declarations and corrections.
3. Freeze the conceptual data model.
4. Choose the backend, authentication, local storage and synchronization mechanism.
5. Confirm the privacy requirements, notifications and detailed acceptance testing criteria.

Earlier confirmed rules and their limited prototype results are recorded in the [worked examples](10-planned-observed-examples.md). They now cover the pattern-default and morning-entry/afternoon-exit interactions. Browser verification of the demonstration remains incomplete and is not production acceptance.

Moving on to development will be the subject of a separate request.
