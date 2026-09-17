# Backend compatibility prototype

Executed September 17, 2026. Disposable source: `/tmp/foylo-backend-spike-oLxx5V`. Reproduction archive: `/Users/leo/.local/share/foylo/archives/2026-09-17/backend-spike.tar.gz`. This archive is stored outside temporary storage and the repository; the working directory in `/tmp` may be cleaned. No production skeleton was added to the repository.

## Question and environment

Can a NestJS controller call an Effect use case through separate core/data workspaces, persist through Prisma 7 on PostgreSQL, roll back a failed write, deduplicate concurrent operations and enforce a real Better Auth session?

Executed with Node 24.14.0, NestJS 12.0.3, Effect 3.22.2, Prisma/client/adapter-pg 7.10.0 and Better Auth 1.7.5. PostgreSQL used a disposable `postgres:17-alpine` container bound only to loopback. The archive includes the exact npm lockfile, minimal Prisma schema, generated migration and assertions. The scratch container was stopped after verification.

The three workspaces compile independently: `packages/core`, `packages/data`, then `apps/api`. Core imports only Effect and owns ports, authorization and replay rules; data implements transaction-scoped repositories; API performs transport/authentication/composition. A generated Prisma type escaping an inferred API return type initially prevented declaration emission. Explicitly typing the composition result fixed the package boundary; compilation and emitted JavaScript execution then passed.

Better Auth is mounted through its Node handler on Nest's Express adapter before JSON parsing. No community Nest authentication wrapper was installed. Nest request authentication resolves the actual database session; household membership is separately checked inside the use case transaction. Email/password is only the local authentication fixture and does not select the product's sign-in methods.

## Executed checks

| Scenario | Result |
| --- | --- |
| Missing session | HTTP 401, no business write |
| Actual auth signup | HTTP 200, cookie and database session created |
| Authenticated account without FamilyMembership | HTTP 403, no Punch |
| Invalid command date | Effect Schema rejects it with HTTP 400 |
| Controller to Effect to Prisma | HTTP 201, one Punch persisted |
| Identical operation replay | Same stored receipt, no duplicate |
| Same operation ID with changed content | HTTP 409, no additional Punch |
| Injected error after Punch insertion | HTTP 500, both Punch and accepted-operation write rolled back; subsequent retry succeeds |
| Six simultaneous identical operations | All return the same receipt; exactly one additional Punch |
| Membership revoked before replay | HTTP 403 even for previously accepted operation |
| Session sign-out | Subsequent request with old cookie receives HTTP 401 |

All 11 passed. Final counts were three Punch rows and three accepted-operation rows. A real Prisma migration was generated and applied to the fresh scratch database. This is not an upgrade-migration test.

## Limits and next gates

The Promise transaction bridge does not yet propagate Effect fiber cancellation. Before reuse, test interruption/timeout while a query is in progress, shutdown and commit outcomes. Revocation completed before a write was tested; a concurrent membership-revocation/write race was not. The minimal schema omits many physical domain constraints and is not a production design. Mobile SQLite, offline sync, Expo session persistence, Apple, email links and production deployment were not tested.

`npm audit` reported four high-severity affected packages in the installed Prisma dependency tree: `prisma`, `@prisma/config`, `deepmerge-ts` and `mysql2`. The production-only audit also reported them. No forced dependency downgrade or untested override was applied. Resolve or document reachability and remediation against the actual production dependency graph before adopting the lockfile. Advisory references: [recursive merge exhaustion](https://github.com/advisories/GHSA-ggr8-5vv4-36mx), [MySQL auth downgrade](https://github.com/advisories/GHSA-3f6p-5ww8-9rcr), [compressed protocol exhaustion](https://github.com/advisories/GHSA-rgwj-5xj2-c3m3).

Verdict: the selected stack and package split work for these local integration cases. This validates feasibility, not a production-ready foundation.

Sources: [Better Auth Express handler](https://better-auth.com/docs/integrations/express), [Prisma adapter](https://better-auth.com/docs/adapters/prisma), [Nest integration alternatives](https://better-auth.com/docs/integrations/nestjs). The executed archive and results are the primary evidence for the table above.

Archive SHA-256: `7badb4c7f8942cf7cf5a1ffec5794c39bf1a93b242eb725babe9bca2eaebcfe3`.
