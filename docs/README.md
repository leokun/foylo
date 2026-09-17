# Foylo: design dossier

Updated September 17, 2026. Current phase: documentation and decisions, no implementation.

These documents draw on the "Family check-in applications" conversation of September 17, 2026, in particular its full brief, its counter-analysis and its final synthesis. Earlier proposals are kept as leads where they have not been confirmed. No new technical, commercial or legal research was carried out for this transcription.

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
11. [Decision register](adr/README.md)

## Statuses

- **Decided**: choice retained in the final synthesis or explicitly confirmed since.
- **To validate**: proposal, design detail or trade-off still open.
- **Rejected/deferred**: option abandoned or postponed, with its reason.

Examples and validation criteria refine the plan; they do not constitute additional validation of the scope.

## Next decisions, in order

1. Confirm the V1 scope and the essential daily journeys.
2. Define the planned/observed rules: meals without check-in, automatic start, corrections, duplicates and changes to a weekly template.
3. Freeze the conceptual data model and the handling of the school calendar.
4. Choose the backend, authentication, local storage and synchronization mechanism.
5. Confirm the privacy requirements, notifications and acceptance testing criteria.

Moving on to development will be the subject of a separate request.
