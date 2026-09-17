# Future scope

## Decided

None of the features below is committed to development. Their deferral follows from the [confirmed V1 scope](01-v1-scope.md).

## Rejected/deferred: outside V1

| Topic | Intent | Reason for deferral |
| --- | --- | --- |
| Rounds | Ordered stops and possible reordering | Validate the daily list of responsibilities first |
| Navigation | Open a destination in Waze or Apple Maps | Complements coordination, not essential to check-in |
| Pricing | Price per meal, duration or time slot | Business rules distinct from quantities and durations |
| Watch | Next task and quick check-in | Additional client and synchronization to be studied |
| Android | Extend mobile access | Focus the first delivery on iPhone |
| Web | Configuration and history | Avoid an additional interface in V1 |
| Multi-household | Blended families and separated parents | More complex sharing and privacy |
| External permissions | Limited access for a nanny or relative | Requires a fine-grained matrix and controlled revocations |
| Rich push and widgets | Contextual actions | Confirm daily usage first |

## To validate

The order of these evolutions will depend on actual usage. No delivery schedule is defined.

Pricing will need to distinguish expected quantity, declared attendance, estimated duration and billing rule. The V1 summary must not promise automatic calculation of an invoice amount.

The envisioned navigation opens one destination at a time; no route optimization or full transmission of a round has been decided.
