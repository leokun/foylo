# Offline operation and synchronization

## Decided

Check-in must work without a network connection. The action is recorded locally and visible immediately, then synchronized when connectivity returns. Check-ins are facts appended to the log.

## To validate

The proposed flow is: input, durable local storage, display, outgoing queue, server validation, then reception of changes on the other devices.

Client-generated identifiers, idempotency keys and an incremental recovery cursor are proposed. Resending the same submission must not create a new fact.

The interface will need to distinguish data that is stored on the device, synchronized, and rejected. Recovery behavior after the application is closed, after an error, and after an account change remains to be specified.

| Situation | Decision required |
| --- | --- |
| Same operation resent | Deduplication by mutation identity |
| Two adults check in the same pick-up | Keep both facts and decide on the business interpretation |
| Two corrections of the same check-in | Define which correction applies and how it is flagged |
| Concurrent modification of a rule | Choose between versioning, resolution or flagging |
| Access revoked before sending | Refuse the server write and define what happens to the local entry |

The append-only log preserves the facts, but does not by itself resolve how they are interpreted. "Earliest wins" or a divergence threshold are unvalidated proposals. Last-write-wins per field for rules and exceptions also remains to be evaluated.

SQLite, PowerSync, ElectricSQL or a dedicated synchronization layer are candidate approaches. No choice of local library or synchronization engine has been made.

## Rejected/deferred

- Waiting for the network to confirm a local check-in: incompatible with the need.
- Confusing the absence of technical duplicates with the absence of duplicate business declarations.
- Designing full autonomous synchronization for the Watch from V1.
