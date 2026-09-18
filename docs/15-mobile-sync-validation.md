# Mobile synchronization: baseline and validation sequence

Updated September 18, 2026. Status: SQLite is the selected local storage family for the next isolated experiment. PowerSync with its native OP-SQLite adapter is the preferred integration candidate, conditional on the gates below. This is not production acceptance or a deployment decision.

## Backend findings executed today

Six assertions ran against a fresh loopback-only PostgreSQL 17 container using the September 17 prototype's locked NestJS/Effect/Prisma environment. The container was stopped afterward. Source and results are preserved outside the repository at `/Users/leo/.local/share/foylo/archives/2026-09-18/sync-gates/`; reproduction uses the original backend archive described in the [backend report](14-backend-compatibility-prototype.md).

| Probe | Observed result | Implication |
| --- | --- | --- |
| Interrupt caller using the original Effect transaction bridge | Transaction continues and inserts a Punch | Caller interruption is not evidence of rollback |
| Revoke after an unlocked membership read, before insertion | Revocation commits, then the previously authorized write commits | Current adapter has an authorization race |
| Lock membership with SELECT FOR UPDATE before writing | Concurrent revocation waits on a database lock until the write commits | Write can be ordered before revocation |
| Revocation completes before locked authorization | Authorization rejects; no Punch inserted | No new accepted write after revocation wins the ordering |
| PostgreSQL statement timeout during pg_sleep, after insertion | Transaction fails and the earlier insertion is rolled back | A database timeout can bound a running query with rollback |
| Wrap the original transaction effect as uninterruptible, then interrupt | Interruption waits until Punch and operation receipt are committed | Bounded completion is a viable policy to investigate further |

All six assertions passed. The first two intentionally reproduce unsafe behavior; six passing assertions do not mean the existing adapter is fixed. The locking and timeout probes isolate Prisma/PostgreSQL behavior, not a corrected end-to-end controller. The interruption probes exercise the original bridge, with a wrapper added only in the last probe. No production code was changed.

Probe source SHA-256: `c4839b3838869f37b30820f67e6f4af944076a45f3f142951c080a222a8de63b`.

## Backend policy for the next experiment

Use a short transaction that commits a Punch and its operation receipt atomically. Once started, allow bounded transaction completion even if the HTTP caller disconnects. A lost response remains an unknown outcome for the client: resend the same operation identity and payload to recover the stored receipt. Do not label a transport timeout as a rejected declaration.

Bound lock acquisition, statements and the complete transaction separately. Keep remote calls outside the transaction. The SQL timeout probe validates one running query, not a complete shutdown budget. The completion wrapper alone does not cancel an underlying query and does not make arbitrary callbacks safe. Track active transactions and drain them before closing the database pool on graceful shutdown. Abrupt process loss, connection loss around commit and graceful shutdown remain required tests.

Authorize writes under a row lock on the current FamilyMembership, held through the business transaction. Revocation must update that same membership row. If a write wins the lock, it may complete before revocation; if revocation wins, the write is refused. Use a consistent lock order for all writers and recheck access before returning stored receipts. This is a proposed implementation contract supported by the two lock-order probes, not a deployed fix. PostgreSQL documents that row locks block conflicting updates until transaction end: [explicit locking](https://www.postgresql.org/docs/17/explicit-locking.html#LOCKING-ROWS).

Re-running the original workspace's production-only npm audit still reports four high-severity affected packages: prisma, @prisma/config, deepmerge-ts and mysql2. Its root manifest and peer dependencies are not a production package graph. No remediation or non-reachability claim has been established. Before reusing these versions, build and audit the actual runtime artifact, trace dependency paths, and verify any compatible remediation without forced overrides. This gate remains open.

## Mobile baseline

The [source-backed research](research/mobile-storage-and-sync.md) recommends PowerSync's native SQLite path rather than maintaining a complete custom replication protocol. Keep NestJS/Effect authoritative for commands and Prisma/PostgreSQL authoritative for accepted facts. Better Auth remains the application session provider; the experiment needs a separate verified-session-to-sync-token bridge.

Prototype the exact local command representation first. Durable local recording, SDK queue insertion and local status must be consistent. A permanent server rejection must leave a durable explanation and unblock subsequent operations. Business conflicts are accepted facts with an unresolved interpretation, not transport failures. Never let a generic SDK update silently replace an accepted Punch or planning revision.

An expired session pauses uploads. Confirmed revocation stops access and triggers removal of sensitive household cache on reconnect. Account switching must isolate storage and pending commands. A device that stays offline cannot be remotely purged. Retention at sign-out and the offline access policy still need explicit privacy requirements.

## Smallest useful next experiment

Use synthetic accounts and a disposable household. Start with an iOS development build, because iPhone is the V1 delivery target; add Android only if it enters the delivery scope. Two isolated native installations must establish cross-device behavior. Expo Go or a browser alone is insufficient evidence for the native adapter.

1. Record a Punch offline, terminate the app, restart offline and verify its identity, planning basis, visible state and queued upload survive.
2. Reconnect through Better Auth and NestJS. Lose the response after database commit, retry, and verify one accepted Punch and a recovered receipt.
3. Upload independent conflicting corrections from both clients in opposite orders. Verify identical unresolved readings with the affected total excluded.
4. Reject one command permanently, terminate around receipt persistence and queue completion, then verify its visible rejection and progress of a later valid command.
5. Revoke membership during queued upload and active download. Verify the write ordering above, measure download cutoff under replication lag, then inspect cache removal and account isolation.
6. Rebuild downloaded state while local work is pending. Preserve unsent operations and the historical planning and fact references they need.
7. Exercise API shutdown and connection loss around commit. Reconcile ambiguous outcomes by operation identity.

The [September 18 native report](16-native-sync-prototype.md) now records a first executed slice on two iOS simulators, including bundled offline restart, command retries, rejection recovery, revocation cleanup and account queue isolation. The full sequence is not complete: physical devices, complete domain interpretation, rebuilds and backend failure boundaries remain pending. A hosted service, billing plan, production migration and deployment are not selected. Service locality, replication permissions, encryption, retention, cost and the dependency audit remain selection gates. If the native command mapping or privacy gates fail materially, evaluate Expo SQLite with a bounded custom protocol against that exact failure.

## Exit criteria

Promote the candidate to a decided synchronization architecture only after the native persistence, command mapping, rejection recovery, idempotency, conflict, revocation and account-boundary gates have evidence, and the deployment constraints are acceptable. Record the result in an ADR then. Meanwhile access/privacy/notification requirements can progress using the explicit offline and revocation limits above.
