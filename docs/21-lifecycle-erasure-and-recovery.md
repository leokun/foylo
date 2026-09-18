# V1 lifecycle, erasure and recovery

Status: adopted product and operational defaults under the delegated decision mandate, September 18, 2026. No deletion or account operation has been executed. These are implementation requirements, not legal retention conclusions or guarantees already provided by a vendor. Verify feasibility and applicable obligations before publishing the policy.

## Separate operations

| Operation | Who can initiate | Accepted shared history | Access and local work |
| --- | --- | --- | --- |
| Archive a Person | Either active adult | Preserved, including historical summaries | End future ordinary planning from a selected effective date; resolve existing exceptions explicitly |
| Leave household | ADULT; OWNER must first transfer or close | Preserved | End membership, reject later writes, purge that household locally |
| Remove the other adult | OWNER | Preserved | Same membership removal semantics; no deletion of the removed account |
| Delete own account | The authenticated User | Retain necessary shared facts with reduced author attribution; linked Person handled separately below | Revoke every session and membership; purge account-scoped local work |
| Erase a Person | OWNER, or the linked adult for their own Person after ending membership | Remove that Person's planning and attendance content, including historical readings | Reject stale references and remove local copies on authoritative reconciliation |
| Close household | OWNER | Entire household enters erasure, including shared history | End all memberships, invalidate invitations/offers, reject all later commands and purge local contexts |

Archiving is the ordinary reversible product action; erasure is a separate destructive action. Both adults can still edit ordinary reference data under the access policy. Neither leaving nor removing a member is a substitute for deleting an account or erasing a Person.

## Confirmation and command ordering

All membership, erasure and closure operations require connectivity and authentication refreshed within five minutes. Show the concrete scope before submission: account, Person or entire household; affected history; other adult's access; and the irreversibility of accepted erasure. Require an explicit confirmation action separate from the ordinary edit/archive control. No offline deletion queue or automatic confirmation.

Before leaving, deleting an account or closing a household, show the initiating device's unsent count and offer Cancel, Synchronize first, or Continue and discard unsent changes. Explain that other devices may have unsent changes that cannot be counted and will also lose authorization. An unacknowledged operation may already have committed. Discarding a local queue is not a server rollback.

Serialize authorization and lifecycle transitions with business writes using the same authoritative ordering. A write ordered before deletion can commit and must then be included in the erasure scope. A write ordered after the transition is rejected. Capture a durable, idempotent lifecycle job and deny affected access in the same transaction; asynchronous cleanup cannot reopen access. Retrying the same request returns its existing status rather than starting another job.

Use states Accepted, Cleaning and Complete for operational tracking. Accepted means access restrictions and a durable cleanup job exist, not that every copy is erased. Show server-side completion only after the applicable active-data cleanup succeeds. State separately that disconnected-device caches and expiring backups follow their own bounds. No cancel or restore action after accepted erasure in V1.

## Account deletion and attribution

An OWNER with another active adult must complete ownership transfer before account deletion, or explicitly select household closure. An owner who is alone closes the household as part of the confirmed deletion flow. Never leave a household ownerless, silently promote another adult or delete the household through a generic account button without disclosure.

Delete credentials, sessions, provider links, verified email, profile, push registrations and account-specific preferences. In an active household retained by the other adult, replace the deleted author's display attribution on accepted facts with Deleted account and sever the account/profile lookup. Keep only a household-scoped opaque author reference needed to distinguish historical actions and conflicts. Do not retain email in Punch payloads or audit descriptions. This reduced attribution is not a claim of anonymization: household context can remain identifying.

If the account is linked to a Person, include that Person's erasure in the confirmed deletion scope after membership ends. Remove their participation records and future Assignments as a responsible adult; remaining responsibilities become unassigned, never silently reassigned. Other Persons' attendance facts authored by the deleted account remain with reduced attribution. The confirmation explains this distinction. Account deletion never implicitly erases a child's Person.

Delete the account's stored support requests, internal notes and feature-request links. Recompute feature counts. Independently written non-personal product requirements may remain, without source quotes or identifying context, under the [support policy](19-support-and-feedback.md#internal-list-and-prioritization). Serialize request acceptance with account deletion so an in-flight submission cannot recreate erased content.

## Person erasure and household closure

Person erasure removes the target's profile, planning rules, exceptions, attendance payloads, related historical versions and derived summaries. Remove their assignments and links without deleting unrelated Persons, shared Places or shared Activities. Shared revisions must be rewritten or partitioned so that deleting a reference does not leave a name or payload hidden in historical JSON. Recompute remaining summaries; show a generic history-removed notice instead of pretending past totals never changed. Do not retain the erased name in that notice.

Erasing the Person linked to an active adult requires ending that membership first. The owner cannot erase their own linked Person while retaining ownership; use transfer and leave, or closure. A child has no User account to revoke. Do not infer authority from being assigned to the child's pick-up.

Household closure erases all household profiles, planning, Punch content, derived data and household diagnostic payloads. It does not delete the other adult's independent User account or unrelated support messages; that adult can still sign in without a household. Both memberships are ended, so either account may later create or join a new household with a new identity. Old commands are never replayed into it.

These operations are explicit exceptions to ordinary append-only journal behavior. Traceability does not justify retaining erased content. Retain only the minimum opaque lifecycle marker needed to suppress stale writes, replay and restored copies, with no names, message bodies or attendance details.

## Offline devices and restoration

For membership removal, account deletion or closure, apply restart-safe purge from the [access policy](17-access-and-privacy.md). A disconnected phone cannot be remotely wiped. Its existing local grant permits access for at most the remaining 24-hour period; expiry locks it, and authoritative denial triggers cleanup on reconnection. Do not claim every device has been erased when the server job completes.

Person erasure while other membership remains active requires a household privacy generation. Atomically advance that generation with the erasure transition. Authoritative access validation returns it independently of the upload queue. A mismatch blocks rendering and upload until the client reconciles erased references, cached revisions, side files and queued commands. Reconcile before opening a fresh sync snapshot. Unrelated pending commands retain their actor and operation identity; commands referencing the erased Person are removed with their sensitive content and represented only by a generic discarded-work notice.

A stale generation cannot authorize a business write. Refresh and reconcile first. Never recreate a missing Person automatically to make an old command succeed. Reinvitation, account recreation with the same email, or a fresh household does not restore old authority. Test cleanup interruption before rendering and before sending any queued command.

Backups must not undo accepted erasure. Maintain a restricted deletion ledger separately from restorable application snapshots, containing opaque target identifiers, scope, acceptance time and completion status. Before exposing any restored database or enabling workers, apply all relevant erasures and session/membership invalidations through the recovery cut-off. If the ledger is unavailable or incomplete, keep the restored service inaccessible. Test a backup taken before deletion and restored afterwards.

## Retention defaults and cleanup targets

These durations are chosen service requirements for implementation and provider evaluation. They are not statutory periods. Do not collect production data until the selected infrastructure can enforce them and the policy review is complete.

| Data | Default bound | Cleanup behavior |
| --- | --- | --- |
| Accepted household history | While the household remains active, unless explicitly erased | No silent age-based truncation; closure and Person erasure override preservation |
| Active-system erasure payloads | Cleanup target within 7 days of accepted request | Access denied immediately; retry failed jobs and alert operators before the target is missed |
| Operational logs | 30 days from creation | No household payloads, credentials, support bodies or email addresses; enforce expiry on copies and exports |
| Operational backups, including point-in-time recovery | Maximum 30 days from capture | No indefinite snapshots or archives; inaccessible to application use; restore applies deletion ledger first |
| Closed support requests | 90 days after closure, and never more than 12 months after submission | Delete content, internal notes and feature links; recompute counts; account deletion shortens the window |
| Open support requests | At most 12 months after submission | Notify assigned support staff before expiry; no silent indefinite extension or promise to retain a feature request forever |
| Deletion ledger and opaque lifecycle markers | Until active cleanup completes and the last potentially containing backup expires, then at most 7 further days | Require confirmed cleanup and backup-expiry evidence before removing the marker |

Long-lived receipts, rejected-command payloads, caches and historical revision blobs are not exceptions to erasure. Purge them with their target scope. Unknown or erased identifiers must remain invalid after marker expiry; identifier reuse is forbidden. Normal offline grant expiry still locks and preserves encrypted pending work as specified in the access policy; it is not a seven-day queue deletion timer.

If cleanup misses its target, record failure and escalate internally without exposing payloads in logs or falsely reporting Complete. Any required retention exception needs its own documented purpose, access restriction and reviewed policy; none is presumed by this design.

## Recovery boundary

Ordinary recovery uses a verified authentication method for the same User. Session recovery never creates a new membership or changes ownership. New sessions obtain a fresh authoritative access grant. Old revoked sessions remain revoked.

V1 provides no support-led ownership override when the original owner's identity cannot be authenticated. The other adult keeps their existing authorized access but is not automatically promoted and cannot reset the owner's credentials. If access to the original identity cannot be recovered through the selected authentication provider, ownership recovery remains unavailable. Do not imply that sending a support ticket will bypass this boundary or promise data recovery.

Apple, Google and email sign-in links are selected in the [authentication specification](05-auth-permissions.md). Automatic matching by verified email is selected in that specification. Backup email and manual recovery are outside V1. Verify normal sign-in recovery and provide clear signed-out guidance to the original sign-in method before authentication contracts are frozen. Do not request identity documents, household exports or children's details as an improvised support workaround.

## Required acceptance checks

All checks are pending implementation and execution.

| ID | Scenario | Expected result |
| --- | --- | --- |
| E01 | Archive Person with history and a future exception | History retained; dated archive; incompatible exception explicitly resolved |
| E02 | Adult leaves or owner removes them during a write | Serialized outcome; history preserved; no later authorized write |
| E03 | Owner attempts account deletion | Transfer or explicit closure required; no ownerless interval |
| E04 | Delete account with linked Person and facts about another child | Own Person erased; other child's facts retain reduced author attribution; no email lookup |
| E05 | Erase child with shared revisions and queued offline facts | No residual child payload in history; unrelated data retained; stale facts cannot recreate child |
| E06 | Privacy generation changes with upload paused | Independent validation blocks stale view and upload; reconciliation precedes fresh rendering |
| E07 | Interrupt cleanup and restart app/server | Durable job/marker resumes; access never restored by partial cleanup |
| E08 | Close household with second adult offline | Server access ends; offline limitation stated; second User account survives; purge on authoritative denial |
| E09 | Restore pre-deletion backup | Deletion ledger applied before access or workers; missing ledger prevents activation |
| E10 | Lose lifecycle acknowledgement and retry | One lifecycle job and consistent receipt; no false completion |
| E11 | Delete account while support submission retries | Content and feature links removed, counts recomputed; retry cannot recreate request |
| E12 | Exercise retention boundaries and failed cleanup | All copies expire under policy; overdue jobs observable; no false success |
| E13 | Owner loses all verified authentication methods | No staff override or automatic promotion; existing ADULT permissions unchanged |

Validate these alongside the [access matrix](17-access-and-privacy.md#acceptance-cases) and [journey criteria](18-v1-journey-acceptance.md). The deletion ledger, privacy generation and retention jobs are required capabilities to design, not an already selected schema or implemented mechanism.
