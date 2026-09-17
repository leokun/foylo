# Authentication and permissions

## Decided

The [conceptual model](02-domain-model.md) establishes FamilyMembership as the household access boundary, independently of represented Persons and Assignments. Better Auth is selected as the authentication library; the detailed permissions matrix and external sign-in methods remain open.

The product owner prefers an authentication library integrated into the Foylo backend with independently hosted PostgreSQL, to avoid dependence on Supabase. Better Auth is selected with Prisma 7 under [ADR 0007](adr/0007-backend-stack-and-package-boundaries.md). The [local prototype](14-backend-compatibility-prototype.md) verifies basic database session creation and revocation; mobile integration and the complete session lifecycle still require validation. This is not a decision to build authentication mechanisms from scratch.

Local testability is a selection criterion: account, session and household-access scenarios should run against disposable local data. External identity-provider and real-device flows still need their own integration checks.

## To validate

The household is the primary access scope. FamilyMembership links User and Family; V1 permits at most one active family per account and two active adult memberships per family. Invitation and role policies must enforce those limits.

A Person shown in the schedule does not necessarily have an account. A grandparent could be designated as a responsible adult without being granted access to the application.

The roles under consideration are OWNER, ADULT and MEMBER. Their exact rights remain to be defined: invitations, household management, schedule changes, check-in, correction and data deletion.

The proposed methods are Apple, possibly Google, and email with a sign-in link. Invitations by link or QR code, expiration, single use, access removal and recovery of the owner account remain to be specified.

Server-side access will have to verify active household membership for every operation. Handling data already present on a disconnected device after revocation must be addressed separately from cutting off server access.

## Rejected/deferred

Multi-household support, sharing a child between households and fine-grained permissions for external caregivers are proposed for after V1. A Grant table was suggested in the counter-analysis; creating it immediately has not been decided.
