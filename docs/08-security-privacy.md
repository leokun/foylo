# Security and privacy

## Decided

Detailed security choices remain to be validated. The domain contains sensitive data for families: people, usual places, schedules and responsibilities.

## To validate: proposed requirements

- No real-time location.
- Hosting in the European Union.
- Short-lived invitations and revocation of server access as soon as a member is removed.
- No first name, place or other sensitive detail in external push content.
- No third-party analytics SDK.
- Systematic household access control and traceability of corrections.

These proposals come from the counter-analysis. They do not describe protections already in place.

## Questions to resolve before implementation

1. Which place and person data is actually necessary in V1?
2. How should the local database, sessions and backups be protected?
3. What does an offline device keep after a revocation, and when is local data deleted?
4. What retention period should be chosen for check-ins, logs and backups?
5. How should the append-only history be reconciled with a deletion request or the closure of the household?
6. How should the owner's departure and the transfer of responsibility for the family account be handled?
7. Which data is allowed in technical logs and diagnostic tools?

## Rejected/deferred

Cross-household sharing and limited third-party access are proposed for after V1. Their implications must not be considered resolved by the OWNER/ADULT/MEMBER roles alone.
