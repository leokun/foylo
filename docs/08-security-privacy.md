# Security and privacy

## Adopted V1 requirements

The [access and local privacy policy](17-access-and-privacy.md) is the detailed baseline. It records design defaults, acceptance cases and release gates; these protections are not yet implemented or accepted.

- No live location, unnecessary identity data or third-party behavioral analytics SDK.
- Application data and operational backups hosted in the European Union, subject to actual provider verification.
- Account-scoped encrypted local storage and secure session/key storage; household caches, pending commands and credentials excluded from transferable backups.
- No household details in notification content, ordinary technical logs or app-switcher snapshots.
- At most 24 hours of local access after authoritative validation. Foreground access checks run independently of upload progress.
- Confirmed membership removal immediately hides protected views and starts restart-safe local cleanup. Offline remote erasure is not promised.
- Account switching preserves a separately locked queue; explicit sign-out removes local data after an explicit loss choice when unsent work exists.

The native experiment proved selected mechanics only. In particular, a paused upload left cached records visible after server revocation until an authoritative refusal triggered cleanup. See the [evidence and limitations](16-native-sync-prototype.md#revocation-is-not-immediate-cache-removal).

## Release gates

Storage encryption, key loss, backup exclusions, physical-device behavior, notification cleanup and interrupted purge require implementation and validation. The prototype's plaintext diagnostic snapshots must not ship.

Define account deletion, Person erasure, household closure, exceptional ownership recovery, log retention, server backup lifetime and post-deletion restore controls before release. Accepted household history is retained during active use, but append-only business history does not define a legal retention period or replace an explicit erasure process.

## Deferred

Cross-household sharing and limited external-caregiver access remain outside V1. The OWNER/ADULT matrix does not settle their future privacy requirements.
