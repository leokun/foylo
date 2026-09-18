# V1 access and local privacy policy

Adopted September 18, 2026 as V1 design defaults under the delegated specification mandate. These are product requirements, not implemented protections, legal conclusions or completed acceptance tests. They refine the open access questions in the [native prototype report](16-native-sync-prototype.md). Production implementation remains a separate step.

## Scope and roles

One User may hold at most one active FamilyMembership; one Family has at most two active adult memberships. A family may have only its owner. The product owner confirmed solo onboarding and use: adding children and planning does not require an invitation, and the second adult may be invited later. Represented Persons and Assignments grant no account access.

Use OWNER and ADULT in V1. Do not introduce MEMBER, child accounts or external-caregiver access. Both adults can coordinate the same household; ownership only adds membership and lifecycle administration.

| Capability | OWNER | ADULT | Offline |
| --- | --- | --- | --- |
| Read household plan, history and summaries | Yes | Yes | Within valid local access period |
| Manage Persons, Places, Activities and planning | Yes | Yes | Queue traceable changes within valid access period |
| Change responsible adult | Yes | Yes | Queue change; assignment grants no new rights |
| Declare, correct, annul or resolve a Punch | Yes | Yes | Queue fact with original identity and planning basis |
| Correct another adult's declaration | Yes, traceably | Yes, traceably | Same history and conflict rules |
| Invite or remove the other adult | Yes | No | No |
| Transfer ownership | Initiate | Accept as recipient | No |
| Leave household | Transfer ownership or use closure flow first | Yes | No |
| Close household | Yes, through a separate confirmed closure flow | No | No |

Removing a Person from future planning archives its current use without deleting historical references. Neither role may silently rewrite accepted facts. Privacy erasure and household closure are separate lifecycle operations, not ordinary journal edits; their rules are specified in the [lifecycle policy](21-lifecycle-erasure-and-recovery.md), with execution and retention enforcement still release gates.

Recheck the actor, current membership, role and referenced household entities for every server command and receipt lookup. Client-supplied household IDs, cached roles, responsibilities and synchronization tokens do not authorize a write. Membership changes and write authorization must share the ordering established by the backend transaction lock. A write that wins that ordering can finish before revocation; one ordered afterward is refused.

## Invitations and ownership

The owner creates an invitation while online for one specified recipient email. Use a high-entropy opaque token, stored server-side as a hash, with a 24-hour expiry. Accept only after authentication with that verified email. A link or QR carries the same invitation and confers no access by itself. No child name, place or schedule appears in the invitation payload or pre-authentication screen.

Allow only one outstanding invitation for the available adult slot. Replacement invalidates the prior invitation. Issuance and acceptance both check capacity and the recipient's existing membership. Acceptance atomically consumes the token and creates the membership, so simultaneous attempts cannot create a third member or two active households. Revocation, expiry and replay fail without disclosing household data. A pending invitation is not a membership.

An owner transfers ownership to the existing active ADULT through an online offer requiring both parties' fresh authentication, no older than five minutes. The offer expires after 24 hours and is cancelled by any intervening membership change. Acceptance atomically swaps roles; there is never an ownerless household. Transfer does not automatically remove the previous owner, who may subsequently leave. An owner cannot use ordinary Leave while still owning the family.

Recover account access through a verified supported authentication method. Do not automatically promote the other adult or grant support staff household access when an owner cannot sign in. The [recovery boundary](21-lifecycle-erasure-and-recovery.md#recovery-boundary) excludes support-led ownership overrides in V1. Apple, Google and email sign-in links are selected in the [authentication specification](05-auth-permissions.md); provider integration, account linking and verified identity recovery remain implementation/release gates.

## Access revalidation and offline period

These numbers are chosen V1 defaults, not measured platform guarantees:

| Parameter | V1 value | Purpose |
| --- | --- | --- |
| Maximum age of successful household access validation | 24 hours | Permit a normal day without network while limiting indefinitely stale access |
| Revalidation interval while foreground and network usable | 30 seconds | Check authoritative access independently of the synchronization queue |
| Access-check request timeout | 5 seconds | Avoid blocking local operation behind a stalled request |
| Dedicated synchronization JWT lifetime | 5 minutes | Limit token reuse; not a substitute for membership checks |
| Fresh authentication for membership/lifecycle changes | At most 5 minutes old | Protect online invitations, removals, ownership changes and closure |

A successful access response is bound to the authenticated User, Family and membership generation. It carries a server issue time and expiry. Only successful authoritative validation renews the local access period. A token refresh, background timer, local check-in, sync checkpoint or cached session does not renew it. A removed and later reinvited membership is a new generation: old queued work cannot regain authority automatically.

Check on launch, foreground return, network recovery and every 30 seconds while foreground and connected. These checks must not wait for the PowerSync upload queue. If a prior access period remains valid, the app can show its local data immediately while checking; expose the last successful validation and unsent-work status. A failed network request uses only the remaining prior period and displays Offline access. With no valid period, show a locked screen until online validation succeeds.

The 30-second cadence plus five-second request timeout is a target for a normally scheduled, foreground client with a responsive access service. It is not a universal 35-second revocation guarantee. Suspension, lost connectivity or an unavailable service can retain offline access until the 24-hour period ends. No refresh may extend that expiry while the service is unreachable.

Derive expiry from the last server time and trusted elapsed-time evidence; never extend it because the wall clock was moved backward. Persist enough evidence for ordinary application restart. If a device reboot or clock anomaly prevents trustworthy elapsed-time reconstruction, require online validation before reopening protected data. Platform support and the user impact of that fallback must be tested on physical iPhones.

## Local access states

| State or event | Visible data and local actions | Upload behavior | Retention |
| --- | --- | --- | --- |
| Valid access period, no network | Cached household data and local edits allowed; stale/sync state visible | Wait for connectivity | Preserve pending work |
| Valid period, successful online validation | Normal authorized household view | Validate each command on server | Preserve accepted history and pending receipts |
| Period expired or trusted time unavailable | Lock household views and editing | Pause | Preserve encrypted pending work for same-account recovery |
| Authentication denied or session revoked | Lock protected views until reauthentication | Pause; no silent retries under another account | Preserve encrypted queue; authentication failure is not a business rejection |
| Typed confirmation that membership was revoked | Immediately hide household data and stop local recording | Stop household uploads | Begin restart-safe purge |
| Generic forbidden response without a confirmed membership reason | Lock affected scope and check access | Pause | Do not discard work based solely on an ambiguous HTTP 403 |
| Permanent validation rejection of one command | Show that command's rejection separately from accepted facts | Persist reason, then complete that queue entry | Keep recoverable rejection while account remains authorized |

Only a typed authoritative membership-removal result for the current User/Family/generation starts the revocation purge. A 401, CSRF failure, malformed request, insufficient role for one action or infrastructure error must not erase the household cache. The prototype's generic 403 handling must be refined before reuse.

An expired local access period does not rewrite event times or turn pending facts into rejected facts. After the same User reauthenticates and the same membership generation is validated, resume with original operation identities. Revalidate business references server-side and surface stale planning conflicts. Do not automatically replay commands from a previous membership generation after reinvitation.

## Revocation cleanup and account boundaries

On confirmed revocation, first make the household inaccessible in the UI and stop issuing commands. Persist a purge-pending marker outside the database being removed. On every subsequent launch, process this marker before opening or rendering that household. A crash must never turn an incomplete purge into restored access.

Stop the synchronization worker and notifications, close all readers and writers, then remove the household cache, associated pending command payloads, receipt details, downloaded attachments and local encryption keys. Database side files and diagnostics containing cached data are part of this cleanup. Keep only a generic local access-removed notice, with no names, places, times, occurrence references or rejected command content. Server facts and audit history are not deleted by member removal.

Validate the granularity of cleanup for the selected SDK. In the single-household V1 an account-specific database can be cleared, but the purge must never clear another account's database. Report cleanup failures without reopening access and retry at launch. Removal is a logical application guarantee, not a claim of forensic erasure or remote wiping of an offline phone.

Switching accounts closes and locks the old account context before opening the next one. Pending work may stay encrypted for that original account; do not display it, reinterpret its actor or upload it under the new session. Bind callbacks to their original account and generation, and discard stale results after a switch. Returning to an account requires successful authentication and a valid household access period before its preserved queue can resume.

Explicit sign-out is different from switching: stop workers, remove session credentials, cancel local reminders and remove that account's local data and keys. If unsent work exists, show its count and explain the loss before completing sign-out. Offer Cancel, Synchronize first when online, or Sign out and discard local changes. Never wait indefinitely for connectivity or silently discard unsent changes. Warn that an unacknowledged command may already have reached the server; signing out does not undo an accepted fact. Online sign-out also revokes the server session; offline sign-out must not claim remote session revocation.

## Data minimization and local protection

Adopt the following V1 product constraints:

- No live location collection. A Place needs a display label and household context; street address, coordinates and route history are outside V1.
- A Person needs a household display name and necessary planning relationships. Dates of birth, identity documents, medical notes, photographs and external contacts are outside V1 unless a separately approved need changes scope.
- Host application data and operational backups in the European Union. Verify actual service locations and subprocessors before selection; this requirement does not claim legal compliance or define the route taken by platform notification infrastructure.
- No third-party behavioral analytics SDK. Diagnostic tooling is separately reviewed and must not receive journal payloads or household details by default.
- Encrypt persistent household data and pending commands at rest, with account-scoped keys in non-migrating platform secure storage. Do not place session credentials or keys in the ordinary SQLite payload or application logs. SQLCipher or another concrete storage mechanism is still an implementation choice to verify with the selected SDK.
- Exclude household caches, queues and keys from transferable device/cloud backups. Server state is the recovery source for accepted records; unsent local changes cannot be recovered after device loss or key loss. State that limitation in the product's pending-work and sign-out flows.
- Hide household content in the app-switcher snapshot on backgrounding. Do not show protected details in notification previews or before access validation. Device lock and key availability must be tested; biometric application locking is not yet selected.

The prototype's plaintext JSON diagnostic snapshots and raw-payload display are synthetic test instrumentation. They must not ship in the production application.

## Notification privacy boundary

External and local lock-screen notification content is generic, for example Open Foylo to review your day. Exclude person names, places, declared facts and activity times from title, body and custom payload. An opaque routing identifier may select an internal view, but never authorizes access. Opening a notification follows the same account, access-period and revalidation rules.

Cancel household reminders when revocation is confirmed or the account signs out. A reminder scheduled while authorized must not outlive the known local access expiry unless renewed after server validation. Another adult's synchronized action may make a local reminder stale; recalculate on sync and foreground. If the phone remains offline, a generic stale reminder is possible and must not be presented as verified attendance.

Push and background execution are optional delivery aids, never the only path to synchronization or access revalidation. The [notification policy](06-notifications.md) specifies categories, opt-in defaults and quiet hours. Provider selection and device validation remain open.

## Retention and lifecycle release gates

Keep accepted planning revisions and Punch history while the household is active; do not silently truncate history to simplify synchronization. Cache minimization must preserve the references required by unsent commands and displayed historical readings. No exact mobile cache window is selected yet.

The [lifecycle policy](21-lifecycle-erasure-and-recovery.md) defines account deletion, Person erasure, household closure, recovery boundaries, retention defaults and post-deletion restore controls. Validate them before release. The append-only journal is not an exemption from a deliberate erasure process. Specify how identity attribution can be removed while preserving the other member's valid history where appropriate. Those product defaults require implementation evidence and policy review; they do not select a legal retention period.

Technical logging defaults to event category, result/error code, duration and an opaque correlation identifier. Exclude names, addresses, full commands, tokens, cookies, invitation links and SQL parameters. Keep operational correlation data only for a stated operational purpose; log/backup retention follows the lifecycle policy; enforcement and approved diagnostic tooling must be verified before any production collection.

## Acceptance cases

All cases below are required and unexecuted against this policy. The earlier simulator evidence covers only parts of their mechanics.

| Case | Required result |
| --- | --- |
| Last access validation at Monday 08:00, phone offline at 17:00 | Read cached plan and record locally; expiry remains Tuesday 08:00 |
| Same phone reaches Tuesday 08:00 without validation | Views and edits lock; unsent work remains encrypted and cannot upload |
| Reauthenticate original account with unchanged membership | Original commands resume with original identities; business validation still applies |
| Change device clock backward or restart the app | No extension of access; trustworthy elapsed-time evidence retained or online validation required |
| Reboot prevents trustworthy elapsed-time reconstruction | Protected views stay locked until online validation |
| Active online client, upload queue deliberately paused, member removed | Independent access check runs; typed removal hides data and begins purge without waiting for upload completion |
| Access service times out at five seconds | No fabricated revocation; only remaining offline period is usable |
| Server returns 401 or a role-specific 403 | Lock/pause appropriately; do not erase pending household commands |
| Stop the app after revocation marker but before database removal | Relaunch completes/retries cleanup before any protected rendering |
| Remove then reinvite the same User | New membership generation; old queued work is not automatically replayed |
| Switch accounts while an old request finishes | Old result cannot appear in or mutate the new account context |
| Sign out with one unsent or ambiguously accepted command | Explicit loss choice; cancel preserves it; discard removes local copy without claiming server rollback |
| Two clients accept the same invitation simultaneously | Exactly one active membership created within capacity; no reusable token |
| Owner attempts ordinary Leave before transfer | Refused; transfer or separately confirmed closure required |
| Ownership offer accepted after membership changes | Stale offer rejected; one owner retained |
| Revoke access with scheduled local reminders | No household details visible; reminders cancelled; expired-period reminders cannot persist |
| Restore application backup on another device | No transferable household cache, pending payload or credentials; accepted data fetched after fresh authentication |
| Inspect production diagnostics and app-switcher snapshot | No household payload, credentials or protected preview |

Next: turn this policy into prototype acceptance checks alongside the [notification checks](06-notifications.md#required-acceptance-checks) and [six V1 journeys](18-v1-journey-acceptance.md). Preserve physical-device and lifecycle release gates explicitly.
