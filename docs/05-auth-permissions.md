# Authentication and permissions

## Decided

The [conceptual model](02-domain-model.md) establishes FamilyMembership as the household access boundary, independently of represented Persons and Assignments. Better Auth is selected as the authentication library; the [V1 access policy](17-access-and-privacy.md) defines permissions and offline access, with Apple, Google and email sign-in links selected for V1.

The product owner prefers an authentication library integrated into the Foylo backend with independently hosted PostgreSQL, to avoid dependence on Supabase. Better Auth is selected with Prisma 7 under [ADR 0007](adr/0007-backend-stack-and-package-boundaries.md). The [backend prototype](14-backend-compatibility-prototype.md) verifies basic session creation and revocation. The [native prototype](16-native-sync-prototype.md) exercises a manually forwarded session cookie on simulators; the official mobile integration and complete session lifecycle still require validation. This is not a decision to build authentication mechanisms from scratch.

Local testability is a selection criterion: account, session and household-access scenarios should run against disposable local data. External identity-provider and real-device flows still need their own integration checks.

## Adopted V1 access defaults

The household is the primary access scope. FamilyMembership links User and Family; V1 permits at most one active family per account and two active adult memberships per family. Invitation and role policies must enforce those limits.

A Person shown in the schedule does not necessarily have an account. A grandparent could be designated as a responsible adult without being granted access to the application.

OWNER and ADULT are the V1 roles. Both coordinate planning and traceable attendance history. OWNER additionally administers invitations, member removal, ownership transfer and household closure. MEMBER and child-account roles are deferred. See the [permissions matrix and lifecycle boundaries](17-access-and-privacy.md#scope-and-roles).

Invitations target a verified recipient email, expire after 24 hours and are consumed once within an atomic capacity check. Ownership transfer requires online acceptance by the other active adult. The [access policy](17-access-and-privacy.md) specifies revocation, account isolation and sign-out behavior. The product owner selected Apple, Google and email sign-in links, without passwords. Google means Google account sign-in, including Gmail users; it does not request access to Gmail messages or contacts. Provider configuration, official mobile integration, account linking and verified recovery still require implementation and acceptance. Exceptional recovery follows the no-override boundary in the lifecycle policy.

Every server operation rechecks authoritative membership. Local access lasts at most 24 hours after successful validation; foreground online checks run every 30 seconds with a five-second timeout. These are adopted requirements, not verified implementation guarantees. Confirmed membership removal triggers restart-safe cleanup independently of the synchronization queue.

## Selected sign-in methods

Offer Continue with Apple, Continue with Google and a sign-in link sent by email. These are product choices, not evidence that provider integrations are configured or tested.

Use one User per verified email address. When Apple, Google or an email sign-in link proves control of the same verified address, resolve to the same User automatically, without a manual linking step. Retain an immutable internal User identifier and provider identity associations; email is the account-matching key, not the identifier for historical facts or memberships. Provider authentication never substitutes for FamilyMembership validation.

The product owner explicitly confirmed the identity model: one internal User can have multiple provider identities, each referencing the immutable User ID rather than the email. Resolve returning Google and Apple sign-ins by their provider identity. Use verified email matching only for initial association, preserving household membership and history when an email changes. Reuse Better Auth's user/account separation; exact table and column names remain implementation details.

Require verified ownership of the incoming address and a verified existing account before automatic association. An unverified signup must not reserve access to another person's identity. Existing provider identities resolve to their already associated User; a later provider email change must not silently move them to another User. Enforce uniqueness and association transactionally, including concurrent first sign-ins. Never merge households or already distinct Users as a side effect of sign-in.

Use the same explicitly tested email comparison policy across sign-in, uniqueness and invitations. Do not infer equivalence by stripping Gmail dots, plus suffixes or comparing display names. An Apple private relay address and a personal Gmail address are different addresses: do not guess that they belong to the same person. Cross-address linking and account merging are deferred in V1; guide the user to their original sign-in method. An invitation must match the verified address actually used, or be replaced by the owner for that address.

No backup email collection, reminder to add one or manual recovery procedure in V1. Normal sign-in through the selected methods remains available. Loss of every supported authentication route does not authorize a support override.

Implementation acceptance must cover Google plus email-link access to one User, verified same-email Apple access, an unverified collision, concurrent sign-ups, returning provider identities, private relay addresses and mismatched invitation addresses. These are required tests, not executed results.

The [Better Auth account documentation](https://better-auth.com/docs/concepts/users-accounts) describes automatic same-email linking under verification or trusted-provider conditions. Foylo requires verified ownership rather than a trust bypass; validate the selected version and configuration. [Apple documents](https://support.apple.com/en-ie/105078) that Hide My Email supplies a distinct relay address. These capabilities support the design but do not prove the local integration.

## Rejected/deferred

Multi-household support, sharing a child between households and fine-grained permissions for external caregivers are proposed for after V1. A Grant table was suggested in the counter-analysis; creating it immediately has not been decided.
