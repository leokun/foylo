# Native synchronization prototype

Executed September 18, 2026. Status: the first native end-to-end slice works on two isolated iOS simulators. Production adoption remains conditional. No physical iPhone, production account, hosted service or production migration was used.

## Evidence and environment

Reproduction archive: `/Users/leo/.local/share/foylo/archives/2026-09-18/mobile-sync-prototype.tar.gz`.

SHA-256: `b43cb8b8ba366ee1a09fdc2c524ca30d2ad0ce2e72a855ee321764cb96bb7d4c`.

The archive contains the mobile and backend sources, npm lockfiles, Podfile.lock, scratch migrations, PowerSync configuration, build logs, native SQLite snapshots, server facts, screenshots and reproduction instructions. `mobile-sync/verify-evidence.py` validates ten assertions against captured evidence; running it alone does not repeat the mobile experiment.

Executed stack: Expo 57.0.23, React Native 0.86.3, PowerSync React Native/common 2.2.1, OP-SQLite 18.2.3, Xcode 27.0 and iOS 27.0 simulators. The backend retains NestJS 12.0.3, Effect 3.22.2, Prisma 7.10.0, Better Auth 1.7.5 and PostgreSQL 17. PowerSync runs locally from the image pinned in the archive. Both service ports bind to loopback. The two synthetic native installations have separate app containers and keychains.

Type checking and native Debug/Release builds passed. Screenshots were visually inspected. The diagnostic interface exposes internal state and is not a product design. Native execution used deep links and an on-screen enrollment control; the evidence helper reads snapshots written by the app after querying its own SQLite database, rather than injecting client database state.

## Executed results

| Scenario | Observed evidence | Boundary |
| --- | --- | --- |
| Local recording and process restart | Exact operation identity, payload, planning-basis fixture and SDK queue survive native termination/relaunch | Debug build; repeated in bundled Release |
| Bundled offline restart | New local command survives termination and relaunch while PostgreSQL, PowerSync and Metro are stopped | API process remains running but its database is unavailable; not an airplane-mode test |
| Restart services | The queued Release command is accepted and appears on the other simulator | Loopback network, synthetic household |
| Lost response after commit | Peer receives the accepted fact while sender retains its pending command; after sender restart, retry recovers the receipt | Exactly one corresponding server Punch and AcceptedOperation |
| Concurrent corrections | Two independently authored corrections reference the same declaration; both devices retain both facts and identify the same unresolved target | Minimal conflict grouping only; complete reducer, resolution workflow and excluded totals are not implemented |
| Permanent rejection and interrupted queue completion | A 422 receipt survives process termination before SDK queue completion; retry clears that entry and accepts a later valid command | Rejection is based on a fixed invalid-occurrence fixture |
| Revocation with pending upload | Authoritative upload returns 403; no corresponding server Punch is created; application clears queue/downloaded rows and sanitizes local receipt payloads | Cleanup occurs after confirmed refusal; no forensic-erasure claim |
| Account switch with unsent work | New account opens a separate local database and does not upload the prior account's command | Accounts share a household; this is queue isolation, not the full household permission matrix |
| Return to original account | Its pending command is restored and later accepted under the original server-authenticated actor | Rapid account switching with arbitrary in-flight failures remains a separate stress case |
| Native session continuity | The Better Auth server cookie survives native restarts in SecureStore and authorizes the NestJS command path and dedicated sync JWT endpoint | Manual cookie forwarding; official Better Auth Expo client, Apple and email-link login remain untested |

Final synthetic server state contained seven Punch rows and seven accepted-operation receipts. The invalid-occurrence operation and revoked queued operation were absent. Extra independent declarations in this fixture are not a validation of Foylo's suspected-duplicate resolution workflow.

The backend copy uses the membership row lock and an uninterruptible transaction completion wrapper investigated in the [backend gate report](15-mobile-sync-validation.md). Its query, lock and transaction timeouts are bounded. This mobile run exercises those paths, but does not repeat the simultaneous lock-order race or establish graceful shutdown and commit-boundary connection-loss behavior.

## Findings that affect the design

### iOS toolchain compatibility

The generated Expo 57 template compiled but initially crashed before rendering under iOS 27 because it did not adopt the scene lifecycle. Applying Expo's documented SDK 57 migration with ExpoReactNativeFactoryProvider and EXExpoAppSceneDelegate resolved the launch. The simulator also required an ad hoc signature with explicit test keychain entitlements for SecureStore. These changes are preserved in the native prototype. Re-running prebuild without preserving that configuration would lose the native adaptation. [Expo scene migration](https://github.com/expo/fyi/blob/main/ios-scene-lifecycle.md)

### Commands and durable receipts

An insert-only PowerSync table carries command intent, while a local-only table records pending and terminal outcomes. The server receives an allowlisted command rather than arbitrary CRUD. Its authenticated actor comes from Better Auth. A downloaded canonical Punch and a local command receipt remain distinct. Permanent rejection is persisted before completing the SDK transaction, which prevents loss of the visible reason at that tested interruption boundary.

The prototype manually forwards the session cookie with the platform cookie jar disabled. A first enrollment attempt failed until the session-token cookie was extracted correctly. This is evidence for this custom native bridge, not certification of the full Better Auth Expo integration.

### Revocation is not immediate cache removal

With a deliberately paused outgoing command, the revoked client still held six downloaded Punch rows in a snapshot taken about 28.5 seconds after membership revocation had committed. This is an observation at one sampled instant, not a measured maximum delay or service-level guarantee. A paused client queue can delay application of downloaded state changes; replication configuration alone is insufficient as a local access-control policy.

Resuming the command through the authenticated backend produced a 403 and triggered explicit local cleanup. A subsequent access check confirmed that the cached rows, pending upload and sensitive receipt payloads were gone. Minimal notices use access-removed status, preserving the distinction between losing access and a business command being rejected. Server history remains intact.

The next specification must define access revalidation on launch/foreground, acceptable online stale-access duration, offline access lifetime, sign-out retention and cleanup recovery. A disconnected device cannot be remotely erased. Periodic revalidation and interrupted-purge recovery were not implemented in this probe, so this gate remains open.

## Remaining gates

- Test two physical iPhones, native suspension and real network loss. Simulator success is not device acceptance.
- Implement the complete conflict reducer, totals exclusions and explicit resolution; exercise both arrival orders. The probe only proves transport preserves both corrections.
- Rebuild downloaded state and evolve the local schema while pending commands exist. Preserve historical planning graphs rather than the fixture's single planning-basis string.
- Define and test multi-household access, bounded online revalidation, offline access expiry, logout retention, interruption during purge and in-flight account switching.
- Test session expiry/renewal, the official Expo authentication integration and eventual product login methods.
- Exercise graceful backend drain, abrupt termination and connection loss around commit. The command's idempotency identity remains the recovery mechanism.
- Review actual runtime dependency artifacts. The disposable backend audit still reports four high-severity affected packages; the mobile production-only npm audit reports ten moderate entries. Neither graph is a production deployment artifact and no exploitability or remediation claim is made.
- Confirm hosting locality, PostgreSQL replication privileges, encryption, backup exclusions, operational monitoring and cost.

## Verdict and next step

Retain SQLite with the native PowerSync adapter as the preferred technical direction. Native persistence, command upload, server idempotency, cross-device replication and the tested rejection recovery are feasible with the selected backend. Do not promote the candidate to a fully decided production architecture yet.

Next, specify access/privacy behavior using the observed revocation delay, then complete the remaining native recovery and domain-reducer checks. This replaces a repeated generic engine comparison with concrete acceptance work.

The test API, Metro process, two disposable containers and two dedicated simulators were stopped after verification. Their source/evidence archive is retained outside the repository. The experiment adds no production implementation or deployment.
