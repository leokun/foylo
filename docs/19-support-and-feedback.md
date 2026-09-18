# V1 support and feedback

Status: inclusion explicitly requested by the product owner on September 18, 2026. The product owner selected storage in the Foylo database and an internal list for feature-request prioritization. The defaults below specify submission and triage; implementation and acceptance remain pending.

## Scope and entry point

Provide a Support and feedback entry in Settings for signed-in users, including users without an active household or whose household access has expired. The form must not require permission to read household data. Offer three categories: Support issue, Question and Feature request.

The form contains a required category, a required subject (1-120 characters after trimming) and a required message (1-5000 characters after trimming). Show the signed-in account as the submitter. Explain that the message will be stored by Foylo for support and product review and advise against including children's names or sensitive family information. No attachments in V1.

Submit only after the user selects Send. Transmit the selected category, subject, message, authenticated account identifier, application version, platform version and an opaque request identifier. Show this metadata list before sending. Do not automatically attach household identifiers, planning, Punch history, child profiles, logs, database snapshots or notification tokens. Staff access to a support request grants no access to the user's household.

## Storage and recovery

Online submission is required. Offline, allow editing while the form remains open and show that sending is unavailable. V1 does not persist drafts to disk or enqueue background submission. Warn before navigation would discard nonempty text; unexpected app termination can lose the unsent draft, which must be stated in the form.

Keep the content visible during a failed submission so the user can retry. Assign a stable request identifier to each submission attempt and reuse it for retries of the same content. The server must store one request for that identifier and return the same receipt on replay. Changed content requires a new identifier. If a response is lost, show reception as unconfirmed and offer retry; do not claim failure means nothing was received.

Show confirmation only after durable server acceptance, with a request reference. Confirmation means received, not resolved or accepted onto the roadmap. No response-time promise, emergency support claim or feature-delivery commitment. Store requests durably in the application PostgreSQL database. Successful submission does not depend on an external mailbox, ticketing service or forwarding job.

V1 collects requests and makes them available for internal review. It does not promise a reply channel or automated response. In-app conversations, user-facing ticket history, chat, attachments, public voting and public roadmap tracking are deferred. The form's existence does not authorize sending any message during specification work.

## Internal list and prioritization

Provide a minimal restricted internal list backed by the same database. The review surface is internal tooling, not a public web client or a household feature. Its implementation form remains open. A household OWNER role never grants reviewer permissions.

Requests have category, subject, message, submitter reference, creation time and a workflow status: New, Reviewing or Closed. Closing records an outcome and optional internal note; it does not imply that a requested feature shipped. Reviewers can filter by category/status/date and search subject and message within their authorized scope.

Maintain a separate feature list with a short title, a reviewed problem description, status (Proposed, Planned, In progress, Delivered or Declined), priority (Unassessed, High, Medium or Low), effort estimate (Unknown, Small, Medium or Large), and a brief priority rationale. These are proposed internal defaults under the delegated mandate. No delivery dates are promised by assigning a priority.

Reviewers can link multiple feature requests to one feature, or unlink an incorrect match. Preserve each original request until its retention deadline; grouping does not silently rewrite it. Each request links to at most one feature in V1. Feature grouping is manual, with no automatic semantic merge.

Show both the number of linked requests and distinct requesting accounts. Repeated requests from the same account do not increase the distinct-requester count. Technical replay increases neither count. Counts cover retained requests, not a permanent historical popularity score; account erasure and retention expiry reduce them. Do not infer household counts by collecting additional family identifiers.

Allow sorting by decided priority, distinct requester count, recency and effort. Keep Unassessed and Unknown explicit rather than treating them as low priority or cheap work. Default feature view shows unassessed items first, then High, Medium and Low, with distinct requester count descending within each group. Frequency informs decisions; it does not automatically outrank data loss, access failures or a blocking support issue. Support issues and questions remain visible in their own request filters even when they have no feature link.

Record reviewer, time and reason for status, grouping and priority changes. Internal notes follow request retention and must not copy family details into the feature list. An independently written, non-personal product requirement may remain after its source requests are erased; remove requester links, quotes and identifying context, then recompute counts. This is not permission to preserve request bodies indefinitely.

## Privacy and operational gates

Use authenticated submission, server-side validation and abuse controls. Do not place message bodies or email addresses in general application logs. Restrict request access to authorized support staff and make the recipient purpose clear to the user.

Before release, assign responsibility for reviewing the internal list, verify database access restrictions, enforce [support-request retention and deletion](21-lifecycle-erasure-and-recovery.md#retention-defaults-and-cleanup-targets), choose abuse limits, and verify the triage workflow. Support content needs its own retention policy; it must not silently inherit indefinite household-history retention. These remain release gates, not implemented capabilities.

## Required acceptance checks

All checks are pending implementation and execution.

| ID | Check | Expected result |
| --- | --- | --- |
| S01 | Open Settings with valid account but no active household | Form available without loading family data |
| S02 | Submit each category with valid fields | One durable request, correct category and receipt reference |
| S03 | Empty/oversized fields or unauthenticated request | Accessible field errors or authentication requirement; no false confirmation |
| S04 | Offline edit, navigation away and send failure | Offline state explained, discard warning, failed content retained for retry |
| S05 | Server accepts but response is lost; retry same request | One stored request and same receipt, no duplicate request |
| S06 | Inspect payload and diagnostic logs | Only disclosed fields sent; no automatic family data or message bodies in logs |
| S07 | Database unavailable or transaction fails | No false receipt; retry preserves submitted content |
| S08 | Open internal list with authorized reviewer and ordinary household owner | Reviewer can triage; ordinary owner cannot access other users' requests |
| S09 | Link three requests from two accounts to one feature; retry one request | Three requests and two distinct requesters; retry changes neither count |
| S10 | Sort/filter features and update priority | Frequency, priority and effort remain distinct; unassessed values visible; edits traceable |
| S11 | Delete an account or expire its requests | Request content and links removed; counts recomputed; retained feature description contains no copied personal data |
