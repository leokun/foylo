# Notifications

## Adopted V1 policy

Status: adopted product defaults under the delegated decision mandate. This specification is not implemented or verified on a device. Provider choice and delivery feasibility remain open.

Notifications are optional aids to coordination. Today, history and synchronization remain usable when permission is denied, delivery fails or every category is disabled. A notification never creates a Punch, proves attendance or acknowledges a business change.

The [access and privacy policy](17-access-and-privacy.md#notification-privacy-boundary) applies to every category. Title, body and custom payload exclude names, places, activity times and declared facts. Example visible text: Open Foylo to review your day. Opening a notification requires the correct account and valid household access before showing details. An opaque routing identifier grants no authority.

## Categories and preferences

| Category | V1 behavior | Initial preference |
| --- | --- | --- |
| Planned responsibility | One local reminder 10 minutes before an explicitly planned drop-off or pick-up assigned to the signed-in adult | Off |
| Shared changes | A generic remote alert after an accepted planning change, declaration, correction, annulment or resolution by the other adult | Off |
| Missing check-in | No automatic reminder for an undeclared school arrival, school pick-up, care visit or lunch | Not offered |

Preferences are per account and device. Enabling one category does not enable the other. Ask for system notification permission only after the adult enables a category. If permission is denied, show its disabled status and a route to system settings without repeated prompts. Changing a preference on one phone does not silently change another phone.

Responsibility reminders require an explicit assignment to the current account's Person. Assignment to a child or a Person without an account never grants access or creates a recipient. For an unassigned responsibility, show it in Today without notifying every adult. The lead time is fixed at 10 minutes in V1. Reminder dismissal changes no declaration or assignment.

Shared-change alerts are best effort, triggered only after server acceptance. Do not alert the author, emit an alert for each technical retry, or include the accepted fact in the payload. Within a household and recipient, coalesce changes over a 60-second window into one generic alert, then start a new window for subsequent changes. Revalidate recipient membership before dispatch. Alert delivery or opening is never a read receipt. While the app is foregrounded, refresh visible state without adding a system banner for that same change.

## Quiet hours and relevance

Quiet hours default to 21:00-07:00 in the household timezone, initially Europe/Paris. Each device can edit or disable them. The interval includes its start and excludes its end; an equal start and end is rejected, with disabling offered explicitly. Quiet hours suppress both categories. Discard suppressed alerts instead of delivering a morning backlog. A reminder at 06:55 for a 07:05 responsibility is therefore skipped.

Use the household's local calendar and timezone when computing reminder instants, including daylight-saving transitions. A phone travelling to another timezone does not move the household schedule. Recompute pending reminders after a household timezone or quiet-hours change. A reminder is eligible only if its trigger is still in the future; do not deliver overdue reminders after launch, permission changes or reconnection.

Do not schedule reminders for cancelled sessions, unresolved planning conflicts or unknown calendar coverage. Show these conditions inside the application. An inferred normal school pick-up, presumed lunch or absent care declaration is never treated as a missing required check-in.

## Scheduling and synchronization

Use one local reminder identity per account, membership generation, occurrence and responsibility kind. Recomputing replaces that reminder rather than adding a duplicate. Different sessions on the same day retain distinct identities. The reminder ledger is delivery bookkeeping, not a source of business facts.

Schedule only within the current local access grant, with the trigger strictly before its expiry. Renewing access permits a refreshed schedule; local use or a notification cannot renew access. Clear scheduled and delivered household notifications when disabling a category, signing out or confirming revocation, and on switching away from an account. An already viewed notification cannot be recalled.

On launch, foreground, network recovery and accepted local or synchronized changes, reconcile reminders with the latest available plan and facts. A cancellation or reassignment removes the old reminder. A changed time replaces it if the new trigger remains eligible. An unambiguous declaration completing that exact responsibility removes its redundant reminder; do not cancel another session's reminder or infer completion across unrelated actions. In particular, a Care exit does not fabricate a school pick-up declaration.

If a different phone records completion, this phone can cancel a pending reminder only after learning the fact. Background execution, silent push and timely delivery are not guaranteed product behavior. An offline phone may still display a stale generic reminder within its valid access period. Explain this limitation in notification settings, without implying that attendance is unverified merely because a reminder appears.

Remote notifications are optional coordination hints. On open, recover state through normal synchronization and access checks. A queued alert can arrive after removal or account switching: its generic content must disclose nothing, and its route must never open the former household. Do not send new alerts after authoritative removal.

## Required acceptance checks

All checks below are pending implementation and device execution.

| ID | Check | Expected result |
| --- | --- | --- |
| N01 | Fresh install; both categories off; permission denied | No prompt until opt-in; every core journey remains usable |
| N02 | Assign 17:00 pick-up to A; recompute twice | One eligible 16:50 reminder on A, none on B |
| N03 | Change time, reassign, then cancel the occurrence | Old reminders removed; only the latest eligible assignment can notify |
| N04 | Other phone completes the same responsibility | After synchronization, redundant reminder removed; offline stale delivery remains possible |
| N05 | No care entry/exit or lunch confirmation | No missing-check-in alert or fabricated attendance |
| N06 | Quiet hours, timezone change, daylight-saving boundary, late reconnect | Correct household-time trigger; suppressed or overdue alerts not replayed |
| N07 | Grant expires before trigger; then renew, switch account or revoke | No reminder beyond grant; correct account isolation and cleanup |
| N08 | Retry accepted command; several changes within 60 seconds | One coalesced alert for recipient, no alert to author, no retry duplication |
| N09 | Lock screen, custom payload, delayed alert after revocation | Generic content only; opening cannot bypass access rules |
| N10 | Background delivery unavailable; app reopened | State recovered through normal sync without dependence on push |

## Technical choices and deferred scope

Expo Notifications and Expo Push remain candidates, not selected providers. Validate local scheduling limits, delivered-notification cleanup, permission transitions, foreground suppression, token lifecycle, routing privacy and quiet-hours enforcement on physical iPhones. Any provider metadata and processing locations require the privacy review described in the access policy.

Rich notifications, action buttons that mutate facts, escalation, mandatory acknowledgements, configurable per-activity lead times and repeated reminders are deferred. Notification delivery is not a safety or attendance guarantee.
