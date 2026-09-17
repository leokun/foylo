# Offline operation and synchronization

## Decided

Check-in must work without a network connection. The action is recorded locally and visible immediately, then synchronized when connectivity returns. Check-ins are facts appended to the log.

## Adopted business behavior

The [V1 business decisions](12-v1-business-decisions.md) define idempotent retries, explicit concurrent-fact resolution, planning revision conflicts and separate event/recording/receipt times. These rules are independent of the synchronization engine.

## To validate: technical integration

The proposed flow is: input, durable local storage, display, outgoing queue, server validation, then reception of changes on the other devices.

Client-generated identifiers, idempotency keys and an incremental recovery cursor are proposed. Resending the same submission must not create a new fact.

The interface will need to distinguish data that is stored on the device, synchronized, and rejected. Recovery behavior after the application is closed, after an error, and after an account change remains to be specified.

| Situation | Decision required |
| --- | --- |
| Same operation resent | Deduplication by mutation identity |
| Two adults check in the same pick-up | Keep both facts as a suspected duplicate until explicit resolution |
| Two corrections of the same check-in | Flag competing corrections, exclude affected totals, require explicit resolution |
| Concurrent modification of a rule | Retain the accepted revision and flag stale edits as conflicting proposals |
| Access revoked before sending | Refuse the server write and define what happens to the local entry |

The append-only log preserves facts. The adopted business interpretation rejects earliest-wins, time-difference thresholds and silent last-write-wins for conflicting declarations or planning edits.

SQLite, PowerSync, ElectricSQL or a dedicated synchronization layer are candidate approaches. No choice of local library or synchronization engine has been made.

## Rejected/deferred

- Waiting for the network to confirm a local check-in: incompatible with the need.
- Confusing the absence of technical duplicates with the absence of duplicate business declarations.
- Designing full autonomous synchronization for the Watch from V1.
