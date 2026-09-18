# V1 specification and implementation sequence

Status: consolidated specification baseline, September 18, 2026. Product requirements and adopted defaults are documented. Production implementation has not started; end-to-end acceptance and the release gates below remain open. This plan does not itself select providers, deploy services or authorize treating prototype code as production code.

## Product contract

Foylo helps two adults coordinate one household on iPhone, then compare planned participation with declarations and explicitly identified inferences. Today is the primary daily entry point. A Person is independent of a User; an Assignment never grants access. A household may temporarily have only its owner, and an account may have at most one active household.

V1 includes people, places, activities, weekly planning, school calendars and closures, dated exceptions, drop-off/pick-up responsibilities, one-action care recording, traceable corrections, monthly summaries with Excel export, optional generic notifications and a support/questions/feature-request form. Android, web, Watch, multiple households, live location, billing, external caregiver permissions and rich notification actions are outside V1.

This document assembles the specification without replacing its detailed sources. Individually confirmed product rules remain authoritative; adopted defaults fill the remaining cases. A material implementation constraint must be recorded and reconciled with those sources before changing behavior.

## Requirement map

| Area | Required behavior | Authoritative detail | Acceptance |
| --- | --- | --- | --- |
| Household and access | Two adult roles, targeted invitation, explicit ownership transfer, account isolation | [Access policy](17-access-and-privacy.md) | Access matrix and J01 |
| Setup and Today | Distinct children, activities, places, responsibility and next action | [Scope](01-v1-scope.md) | J01 |
| Planning | Stable occurrence identity, dated immutable revisions, retained exceptions | [Business decisions](12-v1-business-decisions.md) | J01-J03 |
| Calendars | Official school-vacation and public-holiday imports, academy-to-zone mapping, versioned offline data and explicit precedence | [Business decisions](12-v1-business-decisions.md#calendars-and-exceptions) | J02 |
| Attendance | Pattern-derived care boundary, optional explicit boundary, lunch presumption, no mandatory school check-in | [Confirmed scope rules](01-v1-scope.md) | J04-J06 |
| Corrections and conflicts | Append-only actions, explicit resolution, no arrival-order winner | [Business decisions](12-v1-business-decisions.md#concurrent-declarations-and-corrections) | J04-J06 |
| Monthly comparison | Separate provenance, excluded sessions and incomplete totals | [Journey criteria](18-v1-journey-acceptance.md#j06-compare-a-month-with-missing-data-visible) | J06 |
| Offline and synchronization | Durable operations, original actor and planning basis, retry deduplication, visible pending/rejected state | [Offline policy](04-offline-sync.md), [access policy](17-access-and-privacy.md) | J04-J05 and access matrix |
| Notifications | Device opt-in, generic content, quiet hours, expiry-bounded reminders | [Notifications](06-notifications.md) | N01-N10 |
| Support and feedback | Three categories, explicit online send, database receipt, restricted triage and feature priorities | [Support form](19-support-and-feedback.md) | S01-S11 |

J01-J06 refer to the [six journey acceptance criteria](18-v1-journey-acceptance.md). Requirement coverage is not evidence that these tests have passed.

## Invariants for every implementation lot

- A Punch records a declaration or an explicit action on declarations. Planning and elapsed time never fabricate a Punch.
- No care declaration means normal school arrival/pick-up is assumed, not verified. It creates neither care duration nor an incomplete-care warning.
- Preserve OccurrenceRef, PlanningRevision and PlanningBasis across permitted edits and offline replay. Historical facts and calculations cannot change through an ordinary future planning edit.
- Identical operation replay is idempotent. Independently identified declarations are separate facts, including identical timestamps. Competing facts require explicit resolution.
- The same full fact set yields the same reading regardless of delivery order. Affected unresolved quantities are excluded, never silently zeroed; valid estimates retain their provenance.
- Business completeness, synchronization status and access status are separate. Neither a complete local interval nor a delivered notification proves server acceptance.
- Membership validation, local access expiry and typed revocation operate independently of upload progress. Account switching cannot change an operation's author or leak the previous context.
- Core business contracts do not depend on Prisma, Nest, HTTP, Better Auth or React Native types. Follow [ADR 0007](adr/0007-backend-stack-and-package-boundaries.md).

## Implementation sequence

The lots describe future work. No lot is marked implemented here. Keep changes reviewable and record evidence against the linked criteria. Do not copy the disposable prototypes wholesale into the application.

| Lot | Deliverable | Dependencies | Exit evidence |
| --- | --- | --- | --- |
| L0: Close foundational choices | Login/recovery flow, lifecycle and retention requirements, local encryption strategy, sync acceptance decision, reproducible tooling choice | Current dossier | Written decisions and focused compatibility evidence for the chosen auth/storage path; unresolved release items remain explicit |
| L1: Shared business core | Planning expansion, calendar precedence, occurrence readings, corrections/resolutions and monthly calculations | Current domain baseline; can proceed independently of provider selection once implementation starts | J01-J06 business fixtures, controlled time boundaries, delivery-order permutations and competing-resolution cases |
| L2: Authenticated persistence | API/core/data composition, migrations, household membership, authoritative command acceptance and receipts | L0 auth choices, L1 contracts | Real PostgreSQL transaction, authorization, concurrent retry, commit/ack-loss and shutdown tests; no cross-household access |
| L3: Durable mobile foundation | Selected local store, encrypted queue, account context, access grants, sync adapter and recovery | L0 storage/sync decision, L2 | J04 on two physical iPhones; full access matrix, restart-safe cleanup, schema upgrade and snapshot recovery with pending work |
| L4: Planning and Today | Setup, two child plans, official school-calendar importer, assignments and one-occurrence changes | L1-L3 | J01-J03 through the application on both accounts, unknown coverage and stale edit handling included |
| L5: Declarations and history | One-action care, lunch attendance/absence, corrections, annulment and explicit conflict resolution | L1-L4 | J04-J05 through the interface, original history visible, duplicate and competing resolution paths exercised |
| L6: Monthly comparison | Provenance breakdown, actual/retained times, incomplete totals, history drill-down and Excel export per child or combined, including care/lunch/nanny breakdowns | L1, L5 | J06 numeric fixtures and both-device convergence, timezone and month-boundary checks |
| L7: Notifications | Local scheduling and accepted shared-change alerts using a validated provider | L0 provider review, L3-L5 | N01-N10 on physical iPhones, denied permission and unavailable background delivery included |
| L8: Support and triage | Settings form, database requests, restricted internal list and feature prioritization | L2 identity, reviewer authorization and retention policy | S01-S11, including retries, grouping, distinct requester counts and erasure |
| L9: Release qualification | Complete journeys, operational recovery and distribution preparation | L0-L8 and all release gates | Recorded evidence for every required criterion; unresolved release failures prevent release |

L1 does not need a notification provider or support review tooling. L8 can proceed alongside the planning UI after L2. L7 must consume accepted business outcomes rather than infer them from button taps. Early UI work can use synthetic fixtures, but cannot close L3 or end-to-end journey acceptance.

Detailed SQL schema, transport payloads, screen designs, production package versions and tooling are implementation design outputs. Prepare them within their owning lot rather than presenting prototype choices as approved production contracts.

## Outstanding decisions and release gates

| Gate | Decision or evidence still required | Latest dependency boundary |
| --- | --- | --- |
| G01: Authentication | Integrate selected Apple, Google and email sign-in links; verify account linking, official mobile session handling, refresh and recovery | Before L2 auth contracts are frozen; real-account flows before release |
| G02: Storage and synchronization | Verify actual encrypted native store, nontransferable keys/backups, physical-device lifecycle, full conflict model, schema upgrades and recovery with pending operations | Before L3 exit; SQLite/PowerSync remains conditional |
| G03: Membership enforcement | Independent revalidation, trusted access expiry, typed denial, account isolation and cleanup resumed after interruption | Before L3 exit |
| G04: Backend failure behavior | Safe transaction interruption, lock order, bounded shutdown and ambiguous commit recovery | Before L2 exit |
| G05: Erasure and retention | Implement and review the [lifecycle policy](21-lifecycle-erasure-and-recovery.md): deletion, closure, attribution, retention and restore controls | Requirements before persistence lifecycle contracts are frozen; execution verified before release |
| G06: Hosting and operations | EU application-data and backup location, provider review, cost, backup/restore, diagnostics and dependency advisories | Before production services or data collection |
| G07: Notification delivery | Provider selection, payload privacy, permission and scheduling limitations, token lifecycle and quiet-hours enforcement | Before L7 exit |
| G08: Support operations | Database persistence, restricted reviewer access, feature grouping and priority workflow, retention and abuse limits | Before L8 exit |
| G09: Product acceptance | J01-J06, N01-N10, S01-S11, access matrix and physical-device evidence on a traceable build | Before V1 release |

Requirements work may continue while a device or provider gate is unavailable. Record the exact missing evidence; do not substitute a simulator run, build success or a written policy for its required validation. No arbitrary legal retention duration is selected by this plan.

## Verification strategy and current evidence

Use deterministic core examples for business calculations and convergence; disposable PostgreSQL for actual persistence and authorization; device tests for native storage, keys, account transitions, notifications and lifecycle; complete user journeys for the assembled product. Each layer answers a different question.

Existing evidence comprises [20 domain examples](13-prototype-verification.md), [backend compatibility checks](14-backend-compatibility-prototype.md), [failure probes](15-mobile-sync-validation.md) and a [two-simulator native experiment](16-native-sync-prototype.md). Their reports retain the limitations, including incomplete browser verification, minimal native conflict handling and unverified physical-device behavior. None establishes production readiness.

For each future validation record: criterion ID, exact build/revision, environment, initial data, actions, expected and observed result, retained evidence and remaining limitation. If a criterion fails, identify the affected behavior and owning lot. Re-run impacted checks after a repair, without relabelling unrelated evidence as fresh validation.

## Immediate next work

The [lifecycle policy](21-lifecycle-erasure-and-recovery.md) now specifies G05 defaults and the G01 no-override recovery boundary, with E01-E13 acceptance cases. Verify the selected automatic same-verified-email matching and normal sign-in recovery for Apple, Google and email links before freezing authentication contracts. Backup email and manual recovery are deferred. Validate lifecycle policy feasibility before freezing persistence contracts.

Then prepare focused acceptance work for G02-G04 using the existing isolated experiments. Physical devices and real provider access are prerequisites for their corresponding checks. Starting production implementation remains a separate step; this consolidation changes documentation only.
