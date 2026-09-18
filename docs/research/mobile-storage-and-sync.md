# Mobile storage and synchronization with the selected backend

Research date: September 18, 2026. Status: recommendation for the technical baseline, with mandatory validation gates. No mobile build, synchronization service, database migration or recovery test was executed for this research.

## Recommendation

Use **SQLite through the PowerSync React Native SDK and its OP-SQLite adapter**, with the existing NestJS/Effect/Prisma/PostgreSQL backend retaining command validation and Better Auth retaining application authentication. Treat PowerSync as replication and durable transport infrastructure. Keep all business interpretation in Foylo's domain model.

This recommendation follows the frozen [business decisions](../12-v1-business-decisions.md) and [selected backend](../adr/0007-backend-stack-and-package-boundaries.md). It updates the earlier [engine comparison](offline-sync.md), whose server-side Drizzle assumption is superseded. It is an engineering assessment of responsibilities, not a performance benchmark or production acceptance.

Do not combine `expo-sqlite` and OP-SQLite as competing owners of the same database file. The documented native PowerSync path installs `@powersync/react-native` with `@op-engineering/op-sqlite`; its local SQLite reads, writes and upload connector suit the mandatory offline check-in flow. Expo Go requires a different JavaScript adapter, so it cannot establish native-adapter acceptance. [PowerSync React Native and Expo SDK](https://docs.powersync.com/client-sdks/reference/react-native-and-expo)

## Why this option

| Option | Documented capability | Consequence for Foylo |
| --- | --- | --- |
| PowerSync and OP-SQLite | Native SQLite plus queued uploads through a custom backend connector and server replication | Preferred baseline: less synchronization protocol to maintain; still requires application commands, authorization and rejection handling |
| Expo SQLite and Electric | Persistent SQLite is available separately; Electric provides read-path replication, leaving writes to the application | Requires a SQLite materializer, durable command outbox and reconciliation integration before meeting the same native workflow |
| Expo SQLite and custom synchronization | Transactional persistent local storage; no included cross-device protocol | Fallback if service/privacy constraints rule out PowerSync, with explicit ownership of snapshots, change delivery, recovery and retention |

Electric explicitly leaves write synchronization to the integrating application. Its Expo guide still states that PGlite does not work in React Native. A successful browser implementation therefore does not prove a native durable storage path. Its documented authorized proxy can enforce household read policy, but that does not remove the mobile integration work. [Electric writes](https://electric.ax/docs/sync/guides/writes), [Electric Expo integration](https://electric.ax/docs/sync/integrations/expo), [Electric authorization](https://electric.ax/docs/sync/guides/auth)

Expo SQLite remains a credible custom-protocol foundation: it persists across application restarts. Its asynchronous transaction documentation warns that unrelated concurrent queries can join `withTransactionAsync`; the exclusive variant constrains transaction scope. Web support is marked alpha and needs WebAssembly support and cross-origin isolation headers. These are storage and platform facts, not evidence that a Foylo outbox works. [Expo SQLite](https://docs.expo.dev/versions/latest/sdk/sqlite/)

## Proposed command and receipt contract

The following is a Foylo integration proposal, not an SDK guarantee or a finalized API schema.

1. An offline action retains its immutable operation identity, authenticated account identity, household, occurrence reference, planning basis, declared instant and device recording instant. Confirm local recording only after the durable local transaction succeeds.
2. Use the SDK's upload queue as the transport source of truth. Map its supported operations to a small allowlist of domain commands. Do not expose arbitrary table writes or a generic CRUD API. A local status table may track receipts, but must not become an independent second retry queue.
3. NestJS authenticates the current session. Effect orchestrates the domain operation. A Prisma/PostgreSQL transaction validates current membership, entity ownership, planning references and allowed action, then commits the command outcome and accepted facts together. The actor comes from the verified session, never from a client-supplied actor field.
4. Idempotency is scoped to the authenticated actor and command identity. Retrying the same identity and content returns the stored result; changed content is rejected. Recheck current authorization before revealing a previous receipt or accepting a replay. A revoked user must not recover household details through receipt lookup.
5. Return an accepted or rejected business result only after its database transaction commits. Complete the SDK upload only after the client durably stores the response it needs for recovery. A timeout after commit leaves an ambiguous submission that is retried with the original identity.
6. Downloads bring the server's accepted facts and relevant planning history back to each authorized device. Distinguish locally recorded, awaiting confirmation, accepted, rejected and unresolved business conflict. An empty transport queue alone does not prove every command was accepted.

PowerSync permits custom upload endpoints but expects the underlying database write to complete synchronously before upload acknowledgment with its standard checkpoint mechanism. A response that merely queues server work is insufficient. [PowerSync write contract](https://docs.powersync.com/handling-writes/writing-client-changes)

Transient upload failures remain queued. Acknowledged rejected writes are reconciled against server state. Persist the rejection reason before queue completion and remove its optimistic contribution explicitly; do not depend solely on a future household download, which revocation may prevent. Classify expired authentication separately from permanent business rejection and temporary infrastructure failure. Queue completion after a permanent rejection must allow unrelated later commands to progress. This recovery ordering must be exercised under process termination. [PowerSync validation errors](https://docs.powersync.com/handling-writes/handling-write-validation-errors)

The exact representation of command intent in SDK-managed tables, its atomic local status record, and acknowledgment recovery is a prototype gate. No documented generic CRUD example proves this Foylo-specific command mapping.

## Preserve append-only business semantics

These requirements come from the adopted [V1 business decisions](../12-v1-business-decisions.md), independently of the engine:

- A correction or annulment appends a fact referencing its target. Ordinary client updates or deletions of accepted Punch facts are forbidden.
- Two different operation identities can describe the same real-world event. Keep both as possible business duplicates; transport deduplication must not merge them.
- Competing corrections and resolutions remain visible, exclude affected quantities from aggregates and require explicit resolution. Neither device time nor arrival order chooses a winner.
- Retain the original occurrence reference and planning revision for a late declaration. Download enough historical context to interpret it, including facts referenced by corrections and resolutions.
- A stale planning edit is a conflicting proposal. Do not map SDK patch behavior directly to last-write-wins updates of an accepted planning revision.

A server idempotency transaction and deterministic domain reducer are therefore still required. Replication converging to the same rows is only one prerequisite for devices producing the same business reading.

## Authentication, access revocation and account isolation

Better Auth's Expo integration stores session cookies and cached session data in SecureStore; authenticated requests to a custom native API need the documented cookie forwarding. The cached session is useful for continuity, but is not proof of current household authorization. [Better Auth Expo integration](https://better-auth.com/docs/integrations/expo)

Proposed bridge: a NestJS endpoint verifies the Better Auth session and issues a dedicated short-lived PowerSync JWT. Use the verified user ID as subject and validate audience, issuer and signing keys according to the service configuration. PowerSync documents a maximum lifetime of 24 hours and recommends at most 60 minutes; propose a five-minute lifetime for the experiment, subject to reconnect and refresh testing. Do not assume a Better Auth session cookie is a PowerSync token. [PowerSync custom authentication](https://docs.powersync.com/configuration/auth/custom)

For downloads, filter by membership data and authenticated user identity. Client-selected household parameters can narrow a selection but must never grant access. PowerSync documents membership-based joins/subqueries for this purpose. A membership change reaches the replication service asynchronously, so measure revocation latency under replication lag; a short token lifetime does not independently prove immediate household revocation. Uploads check the authoritative membership within the business transaction, with a defined concurrency ordering against revocation. [PowerSync stream patterns](https://docs.powersync.com/sync/streams/examples), [stream parameters](https://docs.powersync.com/sync/streams/parameters)

Proposed application behavior on confirmed revocation: stop household uploads, stop displaying its downloaded records, purge its local sensitive payloads and retain only a minimal non-sensitive rejection notice where needed. On session expiry, pause uploads and request renewed authentication instead of declaring business rejection. On account switch, close the old account's database and cancel its work before opening another account's database. Never replay the old account's pending commands under the new session.

Retention of unsent entries during explicit sign-out, offline session lifetime, encryption/key loss and backup exclusions remain privacy decisions. A disconnected device cannot be remotely erased; deletion on reconnection is not an immediate revocation guarantee. These limits must be reflected in the eventual product requirements.

## Native and browser boundaries

PowerSync's React Native Web integration uses the separate web SDK, workers and platform-specific initialization; that integration is marked beta. Share domain interpretation and command contracts across platforms, but keep storage and lifecycle adapters separate. A browser receipt cannot validate iOS or Android persistence, and a native receipt cannot establish web durability. [PowerSync React Native Web support](https://docs.powersync.com/client-sdks/frameworks/react-native-web-support)

Make synchronization on launch, foreground return and connectivity recovery the acceptance baseline. Do not promise delivery while the operating system has suspended or terminated the application. Browser acceptance must separately cover reloads, worker startup, multiple tabs, storage clearing and unsupported storage environments. Native acceptance must cover actual application termination and restart with pending work.

## Mandatory validation gates

At the time of this research, all rows below were pending. The later [native prototype report](../16-native-sync-prototype.md) records partial executed coverage on two simulators and the remaining gates. It does not establish full application acceptance.

| Gate | Required evidence |
| --- | --- |
| Native persistence | iOS development builds retain a locally recorded Punch and upload intent after offline termination and restart; repeat on Android if it enters delivery scope |
| Backend command mapping | SDK queue reaches NestJS, authenticates through Better Auth and commits through Effect/Prisma without arbitrary CRUD access |
| Idempotency | Response loss after server commit followed by retries creates one fact; changed payload under the same identity fails |
| Business concurrency | Opposing corrections in both arrival orders yield the same unresolved reading and incomplete aggregate |
| Rejection recovery | Termination before and after local receipt persistence preserves a visible rejection and allows unrelated later work |
| Revocation | A queued upload, active download and concurrent revoke/write race enforce the defined policy; measure cutoff under replication lag |
| Account boundary | Sign-out and account switching never expose or replay another account's cached records or commands |
| Download recovery | Reset/rebuild and schema changes preserve pending work and all references needed by historical Punches |
| Platform separation | Real native lifecycle evidence and a separately served browser test, including web storage limitations |
| Operations and privacy | Confirm PostgreSQL replication permissions, service locality, logs, backups, encryption, retention, monitoring and acceptable cost before production selection |

If PowerSync fails a gate materially, compare the bounded custom-protocol fallback against the failed requirement. That fallback must cover planning, membership changes and removals as well as Punches. It requires a commit-safe change cursor; a plain sequence or timestamp is not sufficient. The [earlier research](offline-sync.md#custom-queue-and-cursor) explains the delayed-commit failure case.

The next concrete experiment is the smallest complete slice: two native clients, one household, one offline Punch, one correction conflict, one permanent rejection and one revoked membership. Hosting selection, production deployment and full application implementation remain separate work.
