# Architecture

## Decided

- Mobile application: Expo, React Native and TypeScript.
- Database: PostgreSQL.
- Backend: NestJS, with Effect for business rules and application use cases.
- Server-side data access: Prisma 7, with PostgreSQL.
- Authentication library: Better Auth.
- Monorepo: separate API, core and data packages, with shared API contracts extracted when needed.
- On-demand planned logic and append-only check-ins.
- Offline-first design.

These choices favor sharing the TypeScript business logic and a relational model suited to the links between households, persons, activities and facts.

## Clean architecture and Effect

Clean architecture is an explicit requirement. The domain owns business concepts and rules; application use cases orchestrate them through inward-facing ports. PostgreSQL, authentication, HTTP, email and mobile storage implementations are infrastructure adapters. Only the composition root chooses and connects concrete implementations.

The product owner explicitly permits Effect in the domain itself. Clean architecture does not require an Effect-free business layer. Effect types, schemas, typed failures and services may express business rules and use cases; the dependency boundary excludes concrete Prisma/PostgreSQL, Better Auth, HTTP and React Native types from business contracts. Effect is selected for core business/application services. The isolated experiment uses Effect 3; its exact production version and interruption-safe database bridge remain to validate.

Proposed test structure follows these boundaries: business examples with controlled clocks and in-memory service layers; database adapters against disposable PostgreSQL; actual auth endpoint flows with local email capture; and separate device/provider integration. A fake repository does not validate SQL transactions, migration behavior or mobile lifecycle handling.

## Local development requirement

The product owner prioritizes easy local testing. The selected backend, PostgreSQL and authentication library must support a reproducible local environment with disposable data, without requiring a cloud project for routine development and integration tests. Local email capture and controlled authentication fixtures should cover routine account and session flows. Real Apple sign-in, email delivery and device integration remain separate end-to-end checks. This is a selection requirement, not a verified capability of a chosen library.

## To validate

The proposed separation comprises a mobile application, an API, a TypeScript domain independent of the interface, a data layer and an API client. Future web and Watch clients must not require immediate development.

The domain would carry schedule expansion, the application of exceptions, reading the log and the monthly aggregates. The rules must be usable locally and server-side with the same results.

Business writes would go through an API or server functions to guarantee authorization and idempotency. Table-level security would serve as a complementary defense. The exact contract remains open.

The selected direction is a Foylo-owned NestJS backend with Better Auth, Prisma 7 and independently hosted PostgreSQL. This avoids dependence on Supabase. A managed PostgreSQL provider remains possible; hosting, synchronization and production monorepo tooling remain open. [ADR 0007](adr/0007-backend-stack-and-package-boundaries.md) records the stack and package boundaries.

The selection will have to compare operational simplicity, data location, backups, compatibility with synchronization and cost. The current capabilities of the solutions will be verified at decision time.

## Rejected/deferred

The fully native iPhone option is not retained in the final synthesis. The argument in its favor was proximity to a future Watch; the TypeScript choice favors sharing the mobile domain. Native integration details remain to be studied.

The repository remains documentation-only. A disposable backend experiment outside the repository is authorized to validate the selected stack; it is not the production application skeleton or database design.

## Package boundaries

```text
apps/api          -> packages/core
apps/api bootstrap -> packages/data -> packages/core
apps/mobile       -> packages/core
optional packages/contracts <- API and mobile adapters
```

`apps/api` contains controllers and the minimal HTTP/authentication/bootstrap wiring. Core owns business rules, use cases, ports and business types. Data implements those ports and owns Prisma schema, migrations and generated persistence types. Controllers never query Prisma directly. Nest injection remains at the application boundary; one managed Effect runtime composes core services through Layers.

A contracts package is justified by shared transport schemas, not by a desire to centralize every type. It must not import Prisma or Nest. Mobile storage remains a separate adapter decision: choosing server-side Prisma does not choose the mobile database library.

## Compatibility evidence

The [isolated prototype](14-backend-compatibility-prototype.md) compiled separate workspaces and passed 11 HTTP/database checks, including rollback and concurrent operation replay. Broader integration and production readiness remain open. Earlier [database comparisons](research/effect-database.md) and [Better Auth research](research/better-auth.md) are historical selection inputs, superseded by ADR 0007 where they describe candidate status.
