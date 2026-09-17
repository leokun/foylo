# 0007: NestJS, Effect, Prisma and separate monorepo packages

Status: Decided. Supersedes the server-side Drizzle selection in ADR 0002 and resolves the candidate status of NestJS, Prisma and Better Auth.

Choose NestJS for the backend, Effect for business rules and application use cases, Prisma 7 for PostgreSQL persistence, and Better Auth for authentication. The product owner's familiarity with NestJS and preference for Prisma's separate schema outweigh Drizzle's closer Effect integration. An Effect adapter around Prisma remains necessary; package versions used in the isolated experiment are not a production lockfile.

Use a monorepo with `apps/api` containing Nest controllers, HTTP/authentication adapters and the minimal bootstrap/composition wiring. `packages/core` owns domain concepts, validation, errors, use cases and repository/transaction ports, with Effect allowed throughout. `packages/data` owns Prisma schema, migrations, generated client, mappings and port implementations. Data depends on core; core imports neither NestJS, Prisma nor data. Controllers delegate business decisions to core. Nest owns outer composition and the application-scoped Effect runtime; Effect Layers connect core dependencies.

Create `packages/contracts` only when shared API request/response schemas are needed by the mobile app and server. Domain types remain in core; Prisma types stay in data. Avoid a generic types package that mixes persistence, transport and domain types. Tooling for the production monorepo and mobile local persistence remain open.

The [backend compatibility report](../14-backend-compatibility-prototype.md) records executed local checks and their limits. The prototype is disposable and does not authorize treating its minimal schema, fixture authentication method or transaction adapter as production-ready.
