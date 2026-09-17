# Effect and database compatibility

> Decision update: [ADR 0007](../adr/0007-backend-stack-and-package-boundaries.md) selects NestJS, Effect, Prisma 7 and Better Auth. Candidate wording below records the earlier comparison. See the [executed prototype](../14-backend-compatibility-prototype.md) for current compatibility evidence.

Checked September 17, 2026. Research and proposed validation only: no dependencies installed or runtime integration tested. [ADR 0002](../adr/0002-technical-foundation.md) retains PostgreSQL and Drizzle; this note does not replace that decision.

## Architectural direction

The product owner explicitly allows Effect in business logic. Domain rules, errors, validation and application services may use Effect. Clean architecture requires independence from concrete infrastructure: business modules must not import Drizzle tables, PostgreSQL clients, authentication implementation types or HTTP framework objects. Infrastructure implements application-owned ports; Effect Layers assemble those implementations. This is a Foylo design recommendation, not a requirement to keep the domain free of Effect.

## Release boundaries

Live npm metadata gives these dist-tag values, which are observations rather than a lockfile:

| Package | `latest` | `rc` |
| --- | --- | --- |
| `effect` | 3.22.2 | 4.0.0-rc.115 |
| `@effect/sql` | 0.52.1 | None |
| `@effect/sql-pg` | 0.53.0 | 4.0.0-rc.115 |
| `@effect/sql-drizzle` | 0.51.0 | None |
| `drizzle-orm` | 0.45.2 | 1.0.0-rc.4 |
| `drizzle-kit` | 0.31.10 | 1.0.0-rc.4 |

Sources: npm registry metadata for [Effect](https://registry.npmjs.org/effect), [SQL](https://registry.npmjs.org/@effect%2fsql), [PostgreSQL](https://registry.npmjs.org/@effect%2fsql-pg), [Drizzle bridge](https://registry.npmjs.org/@effect%2fsql-drizzle), [Drizzle ORM](https://registry.npmjs.org/drizzle-orm) and [Drizzle Kit](https://registry.npmjs.org/drizzle-kit).

Effect 4 is now a release candidate, superseding the earlier beta announcement. Its packages share versions; SQL abstractions move into `effect/unstable/sql`. Unstable APIs have different compatibility guarantees from stable core APIs. Version 3 remains a separate maintained branch. The August changes replaced `pg` inside the version 4 PostgreSQL driver with a native protocol implementation. Do not transfer driver configuration examples between generations without checking them. [Migration guide](https://github.com/Effect-TS/effect/blob/main/MIGRATION.md), [August release recap](https://effect.website/blog/effect-v4-rc-august-recap).

## Practical database options

1. **Effect 3 with the official Drizzle bridge.** `@effect/sql-drizzle` 0.51.0 declares peers `effect ^3.22.0`, `@effect/sql ^0.52.0`, and `drizzle-orm >=0.43.1 <0.50`. These ranges admit the observed latest releases. Its PostgreSQL adapter uses Drizzle's proxy and patches query prototypes to implement Effect. This offers native composition while preserving Drizzle, but introduces bridge-specific behavior requiring focused tests. It is not the same implementation as Drizzle 1's native integration. [Package manifest](https://github.com/Effect-TS/effect/blob/v3/packages/sql-drizzle/package.json), [adapter source](https://github.com/Effect-TS/effect/blob/v3/packages/sql-drizzle/src/Pg.ts).
2. **Effect with conventional Drizzle and `pg`.** Drizzle documents a PostgreSQL adapter backed by a `pg` pool. An infrastructure adapter can translate Promise operations and failures into Effect. This keeps the business API in Effect without depending on the bridge. Cancellation does not automatically stop a Promise-based query, so transaction ownership and interruption semantics require explicit validation. [Drizzle PostgreSQL integration](https://orm.drizzle.team/docs/get-started-postgresql).
3. **Effect 4 RC with native Drizzle 1 RC.** `drizzle-orm/effect-postgres` is documented, and release notes identify native Effect 4 support. This is a promising candidate, not a verified combination: both are prereleases. The guide currently installs untagged Effect packages, which resolve to version 3, alongside Drizzle RC, and still shows `pg` parser configuration. Resolve and test an explicit compatible version set rather than copying that command. [Native integration](https://orm.drizzle.team/docs/connect-effect-postgres), [Drizzle releases](https://github.com/drizzle-team/drizzle-orm/releases).
4. **Effect SQL without Drizzle.** Native SQL composition is available, but removing Drizzle would reopen ADR 0002. Keep this as a fallback comparison, not an implicit decision.

The current bridge package has no npm `deprecated` marker, but it remains a pre-1.0 integration package. Its declared peer ranges exclude Drizzle 1 and Effect 4; this is a concrete generation boundary, not proof of runtime compatibility for every query API. [Published bridge metadata](https://registry.npmjs.org/@effect%2fsql-drizzle/0.51.0).

## Transactions and migrations

Drizzle supports transactions, rollback, savepoints and PostgreSQL isolation configuration. Effect SQL exposes `withTransaction`. For the version 3 bridge, prefer the Effect SQL transaction boundary and prove all repository calls use that transaction's connection; do not assume the proxy implements conventional Drizzle transaction callbacks. [Drizzle transactions](https://orm.drizzle.team/docs/transactions), [Effect SQL contract](https://github.com/Effect-TS/effect/blob/v3/packages/sql/src/SqlClient.ts).

Foylo needs one atomic boundary for household authorization, operation-id deduplication, append-only Punch insertion and the accepted-operation result. Define locking or isolation against concurrent membership revocation, with uniqueness constraints for retries. A transaction alone does not establish that policy. Replay after a lost acknowledgement must return the existing result, while a reused operation identifier with different content must fail. External authentication-provider requests do not belong inside this transaction.

Use reviewed Drizzle Kit SQL migrations as the proposed single migration authority. Effect SQL's presence does not require a second migration history. Drizzle documents separate `generate` and `migrate` steps. [Migration workflow](https://orm.drizzle.team/docs/get-started/effect-postgresql-existing).

## Local and mobile validation

Testcontainers provides disposable real PostgreSQL instances and connection URIs. Propose integration checks for rollback, concurrent duplicate delivery, revocation races, migration upgrades, connection cleanup and date/time mappings. This can satisfy the local-development requirement without a cloud project, subject to a working container runtime. [PostgreSQL module](https://node.testcontainers.org/modules/postgresql/).

For Expo, Drizzle documents `expo-sqlite`, bundled SQL migrations and live queries. That is a separate SQLite adapter, not proof that the PostgreSQL Effect bridge works on Hermes. Expo documents persistent local storage and platform-specific transaction APIs. Keep PostgreSQL drivers server-only; share domain logic and contracts, with distinct mobile persistence adapters. Mobile transaction interruption, restart recovery and bundle compatibility remain untested. [Drizzle Expo integration](https://orm.drizzle.team/docs/sqlite/connect-expo-sqlite), [Expo SQLite](https://docs.expo.dev/versions/latest/sdk/sqlite/).

**Recommendation:** retain Drizzle and use Effect throughout business/application services where useful. Evaluate the version 3 bridge first against a conventional Drizzle adapter, and keep the version 4 native path conditional on a pinned compatibility experiment. No library combination here is yet accepted as Foylo's tested runtime baseline.

## MikroORM alternative raised during review

MikroORM is a credible alternative to reconsider, rather than assuming the earlier Drizzle decision cannot change. Its Data Mapper separates persistence from entities, while its Unit of Work tracks changes and its Identity Map maintains one entity instance per request context. These are useful for object-oriented aggregates, but do not by themselves provide native Effect integration. No official Effect integration was identified in the reviewed integration catalog. [Architecture](https://mikro-orm.io/docs/architecture), [integrations](https://mikro-orm.io/docs/integrations).

A proposed Foylo adapter could expose Effect-based repository ports around MikroORM's asynchronous operations. It must isolate the EntityManager per unit of work and deliberately bridge transaction completion, failures and interruption. Sharing one mutable manager across concurrent fibers is not an acceptable default. The domain may use Effect without importing MikroORM entity metadata or collections. This is an integration design to test, not verified compatibility. [Request context](https://mikro-orm.io/docs/identity-map), [transactions](https://mikro-orm.io/docs/transactions).

Better Auth lists community MikroORM adapters, whereas its Drizzle adapter is documented as a built-in integration. Assess the selected community adapter's supported versions, maintenance and migration behavior before choosing it. Authentication can also keep its own database adapter, but that introduces separate schema ownership to define. [Community adapters](https://better-auth.com/docs/adapters/community-adapters), [Drizzle adapter](https://better-auth.com/docs/adapters/drizzle).

For Foylo's append-only Punch journal and derived readings, explicit queries remain a reasonable fit. MikroORM becomes more attractive if rich mutable aggregates and relationship tracking are an intentional design preference. It is compatible with an Effect-based architecture through an adapter in principle, but there is no demonstrated Effect-specific advantage over Drizzle in the evidence reviewed. Keep it in the comparison; replacement of Drizzle remains an open decision.

## Prisma alternative raised during review

Prisma is also a valid backend candidate. Its generated typed client and migration tooling offer a schema-centered workflow. Better Auth documents a built-in Prisma adapter and delegates generated-schema migrations to Prisma's tooling. [Prisma ORM](https://www.prisma.io/orm), [Better Auth adapter](https://better-auth.com/docs/adapters/prisma).

For the documented Prisma 7 client, interactive transactions provide a transaction-specific client. An Effect repository adapter could wrap asynchronous operations with `Effect.tryPromise`, map database failures to application-owned errors and ensure every operation inside a business transaction uses that client. No official native Effect integration was identified in the reviewed material. Promise wrapping alone does not establish interruption-safe transaction behavior. [Prisma 7 transactions](https://docs.prisma.io/docs/orm/v7/prisma-client/queries/transactions), [Effect Promise integration](https://effect.website/docs/v4/getting-started/creating-effects).

Prisma-generated types stay inside infrastructure; domain types and Effect services remain application-owned. Selecting Prisma would reopen the Drizzle decision and migration authority. It offers a credible schema/client workflow and direct Better Auth integration, but the evidence does not show a stronger native Effect integration than Drizzle's dedicated paths. Runtime compatibility, concurrent retries and transaction rollback still need the same Foylo integration checks. No implementation or selection has occurred.
