# Architecture

## Decided

- Mobile application: Expo, React Native and TypeScript.
- Database: PostgreSQL.
- Server-side data access: Drizzle.
- On-demand planned logic and append-only check-ins.
- Offline-first design.

These choices favor sharing the TypeScript business logic and a relational model suited to the links between households, persons, activities and facts.

## To validate

The proposed separation comprises a mobile application, an API, a TypeScript domain independent of the interface, a data layer and an API client. Future web and Watch clients must not require immediate development.

The domain would carry schedule expansion, the application of exceptions, reading the log and the monthly aggregates. The rules must be usable locally and server-side with the same results.

Business writes would go through an API or server functions to guarantee authorization and idempotency. Table-level security would serve as a complementary defense. The exact contract remains open.

Supabase is an option for authentication and managed PostgreSQL. Hono and Fastify are leads for the API. No provider, server framework, host, monorepo manager or deployment tool has been settled.

The selection will have to compare operational simplicity, data location, backups, compatibility with synchronization and cost. The current capabilities of the solutions will be verified at decision time.

## Rejected/deferred

The fully native iPhone option is not retained in the final synthesis. The argument in its favor was proximity to a future Watch; the TypeScript choice favors sharing the mobile domain. Native integration details remain to be studied.

No application skeleton, dependency, SQL schema or infrastructure configuration is created at this stage.
