# Decision register

This register formalizes the choices retained as of September 17, 2026. Details not yet arbitrated remain in the thematic documents.

| Reference | Decision | Status |
| --- | --- | --- |
| [0001](0001-project-name.md) | Foylo working name | Decided |
| [0002](0002-technical-foundation.md) | Expo, React Native, TypeScript and PostgreSQL; original Drizzle selection | Partially superseded by 0007 |
| [0003](0003-planned-and-observed.md) | Planned computed, observed through append-only check-ins | Decided |
| [0004](0004-offline-first.md) | Local recording before synchronization | Decided |
| [0005](0005-occurrence-references-and-planning-basis.md) | Stable occurrence references, immutable planning basis and computed readings | Decided |
| [0006](0006-clean-architecture-and-effect-boundary.md) | Clean architecture; Effect permitted in the business layer | Decided |
| [0007](0007-backend-stack-and-package-boundaries.md) | NestJS, Effect, Prisma 7, Better Auth and separate monorepo packages | Decided |

The [conceptual model](../02-domain-model.md) is frozen for V1. Physical database design remains outside this specification phase.

## To be validated next

- Detailed acceptance criteria for the confirmed V1 scope and iPhone delivery.
- Hosting and remaining backend/authentication integration gates.
- Local storage and synchronization protocol.
- Permissions, privacy and notifications.

Each upcoming decision will describe its context, the choice, its reasons, its consequences and the alternatives set aside. A later change will explicitly state which decision it supersedes.
