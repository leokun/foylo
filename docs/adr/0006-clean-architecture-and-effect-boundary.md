# 0006: clean architecture with Effect permitted in the domain

Status: Decided for the dependency direction and permitted domain use. [ADR 0007](0007-backend-stack-and-package-boundaries.md) selects the backend stack; exact production versions and integration behavior still need validation.

Foylo must preserve its business rules across backend and mobile implementations and support local testing. Domain and application contracts therefore own the abstractions used by infrastructure; PostgreSQL, authentication, HTTP and device-specific code implement those contracts. Concrete adapters are selected at the composition root, not inside business rules.

The product owner explicitly allows the business layer to use Effect. We reject an artificial requirement that the domain be free of Effect: dependency inversion protects it from concrete infrastructure, while Effect can express rules, typed failures and service requirements. This does not commit every UI component to an Effect runtime, choose an Effect major version, or establish compatibility between database/auth libraries.

See the [architecture](../03-architecture.md#clean-architecture-and-effect), [Better Auth assessment](../research/better-auth.md) and [database compatibility findings](../research/effect-database.md). PostgreSQL remains selected. ADR 0007 subsequently replaces server-side Drizzle with Prisma 7.
