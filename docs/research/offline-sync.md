# Offline synchronization: PowerSync, ElectricSQL and a custom queue

Research date: September 17, 2026. Addresses [Compare offline sync engines against an append-only log](https://github.com/leokun/foylo/issues/5) under the [Foylo V1 specification map](https://github.com/leokun/foylo/issues/1). Status: findings for a future decision, no engine selected and no implementation performed.

## Scope and conclusion

The accepted baseline is Expo/React Native, PostgreSQL, server-side Drizzle, offline check-in and an append-only Punch log. Client IDs, idempotency keys, cursors, rejection UX and EU hosting remain proposals. See [architecture](../03-architecture.md), [domain](../02-domain-model.md), [offline operation](../04-offline-sync.md) and [privacy](../08-security-privacy.md).

**Research assessment:** PowerSync supplies the most complete documented mobile persistence and synchronization path among these candidates. Electric supplies read replication, leaving durable offline writes and local materialization to additional components. A custom queue can fit a narrow check-in workflow but transfers all recovery and incremental-download correctness to Foylo. These are responsibility comparisons, not benchmark results or a technology decision.

All three require application-owned authorization, append-only enforcement, idempotency and interpretation of concurrent facts. No engine can decide whether two adults' independent check-ins represent the same real-world event.

## Responsibility comparison

| Concern | PowerSync | ElectricSQL, current Electric Sync | Custom queue and cursor |
| --- | --- | --- | --- |
| Expo storage | Dedicated React Native SDK with local SQLite. | TypeScript client supports Expo; durable local database integration still required. | Expo SQLite is a possible foundation. |
| Uploads | Persistent SDK queue plus application connector and API. | Application outbox and API, or another framework. | Entire outbox, retry and API contract. |
| Downloads | SDK/service replication and checkpoints. | Shape log, handle and offset; persist materialized data and resume state. | Snapshot, change feed, pagination, cursor retention and reset protocol. |
| Server validation | Application API. | Application API. | Application API. |
| Household reads | Authenticated Sync Streams, separate from write policy. | Authorized proxy sets shape filters. | Authorized download endpoint. |
| Rejections | Server state replaces acknowledged rejected changes; explicit error presentation remains application work. | Application rollback/rejection handling. | Application rollback/rejection handling. |
| Operations | Additional managed or self-hosted replication service. | Additional managed or self-hosted replication service and authorized HTTP path. | Fewer services possible, more protocol code and operational responsibility. |

Sources and qualifications follow below. Operational simplicity is an engineering assessment, not measured implementation effort.

## PowerSync

The [React Native/Expo SDK](https://docs.powersync.com/client-sdks/reference/react-native-and-expo) provides SQLite and a connector whose `uploadData()` sends queued operations to a chosen backend. The native adapter requires a native build; Expo Go uses a separate JavaScript adapter. Production feasibility must be checked on actual iOS and Android builds. PowerSync also handles stream subscription and local query updates.

Its [client architecture](https://docs.powersync.com/architecture/client-architecture) stores writes in SQLite and queues uploads. This removes the need to build the basic persistent outbox. It does not establish Foylo's business acceptance state. Distinguish queue completion, server acceptance and receipt of the authoritative replicated row.

The [write contract](https://docs.powersync.com/handling-writes/writing-client-changes) expects backend changes to be committed synchronously before upload completion with the standard checkpoint mechanism. A generic API response meaning only “queued elsewhere” is insufficient. [Conflict documentation](https://docs.powersync.com/handling-writes/handling-update-conflicts) explicitly requires idempotent backend operations because delivery can repeat. For Punch, the proposed backend should accept insertion and correction-as-insertion, rejecting ordinary update/delete commands. Do not inherit generic last-write-wins semantics for facts.

For [validation errors](https://docs.powersync.com/handling-writes/handling-write-validation-errors), transient failures retain queued work. Acknowledged but rejected mutations disappear from the authoritative local view after reconciliation. The documented pattern returns a successful transport response with an explicit rejection result, or replicates rejection details separately. Foylo must durably record that outcome before completing the queue item so it can explain a rejected check-in after restart. Returning a permanent error forever can block subsequent uploads. Authentication expiration and revoked access require separate classification; neither should be disguised as an accepted business write. This proposed integration needs testing when a revoked client cannot download any more household data.

[Sync Streams and RLS](https://docs.powersync.com/integrations/supabase/rls-and-sync-streams) protect different paths: download filters do not authorize uploads. If Supabase is chosen, RLS and stream access rules must agree. With a custom API, active membership checks remain mandatory regardless of database credentials. A client-supplied household ID is not authority.

## ElectricSQL

Current [Electric write documentation](https://electric.ax/docs/sync/guides/writes) states that Electric synchronizes reads from PostgreSQL and does not supply write-path synchronization. Its examples progress from online calls to persistent optimistic state and a local database change log. The documentation warns that the example rollback strategy can clear all local writes. These examples must not become Foylo's rejection policy without deliberate redesign.

The official [Expo integration](https://electric.ax/docs/sync/integrations/expo) supports the TypeScript client. Its native lifecycle integration pauses ShapeStreams in the background and catches up on foregrounding. The same page says PGlite does not yet work in React Native. Therefore, a browser PGlite example is not proof of a durable Expo implementation. A SQLite materializer, durable outbox and reconciliation layer remain integration work unless a separately evaluated framework provides them.

The [HTTP protocol](https://electric.ax/docs/sync/api/http) supplies initial shape loading, incremental offsets and handles, with `must-refetch` recovery when a shape must be rebuilt. Proposed client requirement: atomically persist applied changes with their matching handle/offset; after reset, replace downloaded state without losing pending local Punches. A memory-only React query is insufficient for offline restart.

The [auth guide](https://electric.ax/docs/sync/guides/auth) recommends a proxy that validates credentials and chooses shape parameters server-side. The proxy must constrain table, columns and household filter, including initial and incremental requests. Foylo should check current membership rather than trust stale client parameters. Revocation must also cover already authorized long polls and caches; exact cutoff latency requires testing. Writes use the separate business API with the same membership policy.

## Custom queue and cursor

[Expo SQLite](https://docs.expo.dev/versions/latest/sdk/sqlite/) persists databases across restarts and exposes transactional APIs. That is a storage primitive, not a synchronization engine. The following is a proposed technical design for comparison, not an accepted schema:

1. Commit a client-generated Punch identifier, its optimistic local record and an outbox command in one local transaction. A retry reuses the command identifier and payload.
2. Send commands to an authenticated API. In one PostgreSQL transaction, validate current membership and referenced household entities, record the idempotency outcome and insert an accepted Punch. Reuse of a key with a different payload must fail explicitly. Scope receipt access to the authenticated principal and household.
3. Retry timeouts and temporary failures with bounded exponential backoff and jitter. A timeout after server commit remains ambiguous until an idempotent replay returns the stored result. Preserve unknown failures for diagnosis; do not silently discard them.
4. Persist permanent rejection and remove its optimistic contribution to the accepted view. Retention of rejected payloads, user repair actions and dependent corrections are product/privacy decisions still open.
5. Download authorized incremental changes and advance the cursor in the same local transaction as applying them. Define snapshot boundaries, pagination, expired-cursor reset and account isolation.

**Critical cursor trap:** an ordinary increasing PostgreSQL sequence is not a safe committed-change watermark. [PostgreSQL sequences](https://www.postgresql.org/docs/current/functions-sequence.html) allocate values independently of transaction rollback. Inference: transaction A can allocate 10, transaction B allocate and commit 11, and a reader advance to 11 before A commits. Fetching only IDs greater than 11 then misses A. Candidate remedies include a serialized per-household change counter held through commit, or a correctly consumed committed change stream. They require implementation and concurrency tests; wall-clock timestamps are not an adequate shortcut.

Even though Punches append, memberships, schedules, exceptions and deletion requirements do not disappear. A Punch-only cursor leaves those datasets stale. The custom protocol must include their updates and removals, or explicitly define a consistent refresh mechanism. Retention and tombstones must accommodate devices returning after long absences.

## EU locality and cost

Prices below are published USD amounts observed on the research date, not a Foylo quotation. They exclude the source database, business API, authentication, backups, taxes and engineering time.

| Option | Published service pricing | Locality and unresolved points |
| --- | --- | --- |
| PowerSync Cloud | Free: 2 GB/month synced, 500 MB hosted, 50 peak clients; inactivity deactivation after one week. Pro starts at $49/month, including 30 GB synced, 10 GB hosted and 1,000 peak clients. | An EU region is documented. Confirm location of replicas, support access, logs and backups before accepting an EU-only requirement. |
| Electric Cloud | PAYG: $1 per million writes and $0.10/GB-month retention; bills under $5 waived. PostgreSQL incremental shape-log writes have an additional $2/million charge. Reads/egress/fan-out are listed as free. | Region selection exists, but the reviewed pages do not establish EU-only processing and cache residency. Obtain explicit confirmation. |
| Self-hosted engines | Infrastructure and operations are additional costs; verify the applicable edition/license before choosing deployment. | EU deployment is possible to design, but all storage, proxies, telemetry and backup paths need review. |
| Custom | No sync vendor fee; no defensible fixed monthly estimate without workload and host choices. | Can share an EU API/PostgreSQL deployment; local backups, diagnostics and all external services still need review. |

Sources: [PowerSync pricing](https://powersync.com/pricing), [PowerSync EU region](https://docs.powersync.com/configuration/source-db/security-and-ip-filtering), [PowerSync self-hosting](https://docs.powersync.com/intro/self-hosting), [Electric pricing](https://electric.ax/pricing), [Electric Cloud usage](https://electric.ax/cloud/usage), [Electric deployment](https://electric.ax/docs/sync/guides/deployment).

Electric bills emitted shape-log writes, not simply application check-ins. Initial loads, overlapping shapes and other entities affect quantities. PowerSync's usage axes likewise require estimating history size, initial synchronization, change fan-out and connected devices. Household count alone cannot establish the cheaper option.

## Shared safety and validation gates

These are proposed acceptance checks for a later authorized prototype:

- Kill the application after the durable local write, after server commit but before response, and while applying a download. Restart must preserve pending work without duplicate accepted facts or skipped changes.
- Submit identical operation IDs repeatedly, then the same ID with changed content. Separately submit two independently identified check-ins for the same activity: preserve both facts until product rules decide their meaning.
- Reject an invalid Punch while later valid work is pending. Preserve a visible rejection without indefinitely blocking unrelated work. Exercise corrections referencing pending or rejected facts.
- Revoke membership with an upload queued and a download open. Verify both server paths stop access within a defined bound, then test sign-out and account switching without cross-account replay.
- Restore from an expired cursor, reset a shape, change a local schema, and reconnect after a long offline period. Preserve pending commands while rebuilding downloaded data.
- Test foreground recovery on physical iOS/Android devices. Do not promise immediate synchronization while the OS has suspended or terminated the app.

No server can erase a disconnected device remotely. Local data deletion on reconnection, offline session lifetime, encryption, backup exclusion and rejected-entry retention remain explicit privacy decisions. Append-only business history also does not settle household deletion or retention policy.

Next decision: choose which responsibility profile to prototype once product rules and privacy requirements are resolved. This research did not benchmark performance, provision services, validate a production security boundary, or run the recovery tests above.
