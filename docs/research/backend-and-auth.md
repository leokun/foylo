# Backend and authentication research

Research for [Evaluate Supabase versus a dedicated API for auth, Postgres and business writes](https://github.com/leokun/foylo/issues/6), part of the [Foylo V1 specification map](https://github.com/leokun/foylo/issues/1). Primary sources checked September 17, 2026. Status: research and recommendations, not an architecture decision or implementation.

## Scope and conclusion

The [architecture](../03-architecture.md) already selects Expo, TypeScript, PostgreSQL and Drizzle. It leaves the provider, API runtime and synchronization contract open. This note compares Supabase Auth and Postgres with either Supabase Edge Functions or a dedicated Hono/Fastify API.

**Proposal:** retain Supabase Auth and Postgres as a credible managed base. Keep TypeScript business rules behind a single command API and preserve PostgreSQL transactions for authorization, deduplication and writes. Evaluate Hono on Edge Functions as the simplest deployment, with a dedicated Node API as the alternative if regional processing requirements or synchronization integration cannot be satisfied. Do not select Edge Functions solely because the database is in the EU.

No runtime benchmark, restore exercise, authentication flow or RLS policy has been executed. Provider documentation establishes capabilities, not Foylo's operational readiness.

## Options

| Option | Strength for Foylo | Constraint or cost | Assessment |
| --- | --- | --- | --- |
| Supabase Auth/Postgres + Hono Edge Functions | One provider, TypeScript command boundary, Drizzle support | Runtime limits, pooled connections, regional routing requires validation | Candidate for a small pilot |
| Supabase Auth/Postgres + Hono on Node | Similar HTTP model, conventional process lifecycle and database pool | Additional deployable, hosting bill and operational ownership | Strong alternative for region-controlled business processing |
| Supabase Auth/Postgres + Fastify on Node | Schema-based validation and serialization, Node ecosystem | Separate host and framework-specific adapters | Equally viable if its conventions are preferred |
| Supabase Auth/Postgres + database RPC only | Transactional commands close to data | More business logic in SQL, less reuse with the mobile TypeScript domain | Use selectively for atomic operations, not as the default location for shared domain rules |

Hono documents both [Supabase Functions](https://hono.dev/docs/getting-started/supabase-functions) and a [Node adapter](https://hono.dev/docs/getting-started/nodejs). Migration still requires adapting startup, configuration, pooling and runtime-specific dependencies. Fastify documents JSON Schema validation and response serialization; neither framework by itself proves household authorization or retry safety. [Fastify validation](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/).

## Authentication and household authority

### Verified provider capabilities

Supabase supports email magic links and email OTP through `signInWithOtp`; email templates determine the flow. Account creation can be disabled for a sign-in attempt with `shouldCreateUser: false`. Redirect allowlists and a tested mobile link flow are needed for magic links. [Passwordless email](https://supabase.com/docs/guides/auth/auth-email-passwordless).

Native Sign in with Apple is documented for Expo, using an identity token. Apple's six-month OAuth secret rotation applies to the web/OAuth configuration, not a native-only integration. [Supabase Apple integration](https://supabase.com/docs/guides/auth/social-login/auth-apple).

Apple guideline 4.8 requires an equivalent login option satisfying its privacy criteria when a third-party/social login establishes the primary account, subject to its exceptions. It does not mandate Apple by name. An application using only its own account system is an exception. Guideline 5.1.1(v) requires in-app account deletion when account creation is supported. [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/).

The default Supabase email service is for testing: delivery is restricted to project team addresses and currently limited to two messages per hour. Real-user email sign-in needs custom SMTP or a separately validated delivery integration. Buying Pro does not automatically configure production email; SMTP is a separate operational dependency, not a Free-plan-only restriction. [SMTP documentation](https://supabase.com/docs/guides/auth/auth-smtp).

Supabase exposes an administrative user invitation API. That invitation authenticates/onboards a user; it does not establish Foylo's household membership, role or capacity rules. [Invite user API](https://supabase.com/docs/reference/javascript/auth-admin-inviteuserbyemail).

### Proposed application contract

Follow [authentication and permissions](../05-auth-permissions.md): `User` identifies an account, `Person` identifies someone in a schedule, and `FamilyMembership` authorizes household access. Assigning a grandparent as responsible for an activity must not grant them application access.

Proposed invitation behavior, requiring product confirmation:

1. An authenticated adult with the eventual invitation permission requests an invitation. The server selects the family and allowable role; a supplied family ID alone grants nothing.
2. Store a hash of a random, short-lived, single-use token, its household, issuer, expiry and revocation/acceptance state. Keep the raw token out of logs. Decide whether it is bound to a recipient identity or transferable through a QR/link.
3. The recipient signs in first, including an already-existing account. Household acceptance is separate from account creation.
4. One transaction locks/consumes the invitation, checks current issuer authority and household capacity, rejects expiry/revocation, and creates the membership. Concurrent acceptance and retries must not create extra members.
5. Membership and role changes are server-only commands. User-editable profile metadata and stale JWT claims are not membership authority.

OWNER, ADULT and MEMBER rights remain undecided. Do not invent an implementation permissions matrix or grant an invited adult OWNER automatically. Owner transfer, account recovery, departure and deletion require explicit journeys.

### Revocation and offline access

Supabase access JWTs remain usable until expiry unless the application performs an additional live check; its documentation describes checking `session_id` against `auth.sessions` when sign-out must invalidate access immediately. The default JWT lifetime is one hour. [Sessions](https://supabase.com/docs/guides/auth/sessions).

**Foylo implication:** validate active household membership on every server read and write, including synchronization endpoints. A removed member with an otherwise valid identity token must fail subsequent authorization checks. Household removal does not inherently terminate the identity session, and waiting for JWT expiry does not fix a missing membership check. Coordinate revocation and sensitive writes transactionally so a concurrent command has a defined authorization order. Specify treatment of already-running requests and long-lived sync subscriptions.

A disconnected device cannot be remotely erased immediately. Its local retention, reconnect purge and rejected queued-command experience remain separate requirements in [security and privacy](../08-security-privacy.md).

## RLS and the command boundary

PostgreSQL RLS is default-deny after activation when no applicable policy exists. Owners normally bypass it; superusers and `BYPASSRLS` roles bypass it. Referential-integrity checks can reveal information through constraint failures. RLS therefore complements application authorization only when the actual database role is subject to it. [PostgreSQL row security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html).

Supabase recommends RLS for exposed tables and warns that service keys bypass it. Membership helpers implemented with `SECURITY DEFINER` require carefully limited privileges and an appropriate private schema; user metadata is unsuitable for authorization. [Supabase RLS](https://supabase.com/docs/guides/database/postgres/row-level-security).

**Proposed enforcement:**

- Scope all household data through active membership. Prevent cross-household references through constraints and server validation, not just a top-level request check.
- Deny direct client mutation of memberships, roles and business tables where commands are authoritative. A permissive table-write policy would allow clients to bypass command validation.
- For ordinary API transactions, use a restricted role and transaction-local verified identity context. Do not run every business write with a service key or an owner connection while claiming RLS protects those writes.
- Reserve privileged credentials for tightly scoped administration. Check cross-household access using the production-equivalent database role.
- Test any membership helper against recursion, privilege escalation and identity leakage across pooled requests.

Database functions default to `SECURITY INVOKER`; definer functions need a controlled `search_path` and restricted execution grants. They can encapsulate atomic commands, but moving authorization into a privileged function creates a security boundary that must be reviewed explicitly. [Database functions](https://supabase.com/docs/guides/database/functions).

## Transactions, Drizzle and idempotency

Edge Functions currently allow 256 MB memory and two seconds of CPU per request, excluding asynchronous I/O. Worker lifetime is 150 seconds on Free and 400 seconds on paid plans, with a separate 150-second request idle timeout. These are ceilings, not performance targets for mobile synchronization. [Runtime limits](https://supabase.com/docs/guides/functions/limits).

Supabase documents authenticated Edge handlers with an RLS-scoped client and a separate privileged client. JWT verification authenticates the caller; it does not authorize a household command. [Function authentication](https://supabase.com/docs/guides/functions/auth).

Drizzle supports Supabase through `postgres-js`; prepared statements must be disabled with transaction pooling. A long-lived Node API can use direct connections or a session pooler, subject to networking and connection limits. A dedicated API has host, memory and timeout limits too, even though the Edge CPU ceiling does not apply. [Drizzle Supabase integration](https://orm.drizzle.team/docs/connect-supabase), [Supabase connection modes](https://supabase.com/docs/guides/database/connecting-to-postgres).

PostgreSQL `ON CONFLICT` provides an atomic insert/update primitive under concurrency. It does not define application retry semantics or ensure two independently generated IDs represent the same real-world event. [PostgreSQL INSERT](https://www.postgresql.org/docs/current/sql-insert.html).

**Proposed command semantics:** generate an operation ID before the mobile device queues a write; retain it across retries. In one database transaction, check current membership, validate the command, reserve a unique operation key, append the fact and persist the result needed for replay. Bind the key to its actor/household/operation and a payload fingerprint. The same key and payload returns the prior outcome; the same key with a different payload is a conflict. Return replayed results only after current authorization succeeds.

Use an event ID as the operation ID where appropriate, or a separate command receipt table when one command has several effects. Do not silently update an append-only fact on conflict. Business duplicate rules, such as two adults checking in the same person, remain distinct from network retries and depend on the planned/observed rules. Notifications need a transactionally persisted outbox if committed writes must reliably produce them; an HTTP response and a later push call are not one transaction.

## EU residency and processing

Specific Supabase EU database regions include Paris, Frankfurt, Ireland and Stockholm. London and Zurich are European locations outside the EU. The provider explicitly warns that the general Europe grouping is not a jurisdiction guarantee. Select a specific region if the EU hosting proposal is confirmed. [Available regions](https://supabase.com/docs/guides/platform/regions). Auth records live in the project's PostgreSQL `auth` schema. [Auth architecture](https://supabase.com/docs/guides/auth/architecture).

Edge execution defaults to a region near the caller. A client region option, `x-region` header or documented query parameter selects a region; explicit routing disables automatic rerouting during an outage. The response exposes `x-sb-edge-region`. [Regional invocation](https://supabase.com/docs/guides/functions/regional-invocation).

**Unresolved constraint:** asking the official client to send a region is not proof that all callers, webhooks and direct requests are prevented from executing elsewhere. A regional guard inside a handler already executes after routing. Before promising exclusively EU business processing, establish a provider-supported enforcement mechanism and review ingress/log handling. Otherwise prefer a dedicated API deployed in an explicit EU location for that boundary. Railway documents Amsterdam as an available deployment region, but its service-region selector alone likewise says nothing about every supporting service. [Railway regions](https://docs.railway.com/deployments/regions).

Supabase's DPA describes storage and primary processing in the instructed region with exceptions, and permits subprocessors subject to its transfer provisions. This is not an unconditional EU-only processing promise. Email, support, diagnostics, backups and any future synchronization service require their own data-flow review. [DPA, clauses 6 and 12](https://supabase.com/legal/customer-resources/data-processing-addendum). This research does not establish legal compliance.

## Backups and cost at family scale

Verified list prices in USD, before tax, exchange rates and optional services:

| Scenario | Published base | Planning interpretation |
| --- | --- | --- |
| Supabase Free | $0; two active projects, 500 MB database; inactive projects can pause after a week | Prototype only under the proposed reliability expectations |
| Supabase Pro, one Micro project | From $25/month; $10 compute credit covers one Micro; seven days of daily backups | Baseline budget for real household reliance |
| Pro plus another Micro project | Approximately $10/month extra | Staging is not automatically included in the first-project price |
| Dedicated API on Railway Hobby | $5/month minimum including $5 resource usage; usage above that is extra | Illustrative personal-pilot floor: about $30/month including Supabase Pro |
| Railway Pro instead | $20/month minimum including usage credit | About $45/month combined base if this plan is required |

Sources: [Supabase pricing](https://supabase.com/pricing), [Railway plans and metering](https://docs.railway.com/pricing/plans). These are provider floors, not a measured Foylo bill. Email delivery/domain, backup storage, observability, synchronization products and excess usage are excluded.

Supabase Free requires independently managed exports for recovery. Pro retains daily backups for seven days. Database backups exclude Storage API objects; restores cause downtime. PITR is an additional paid feature requiring at least Small compute, with seven-day retention advertised from $100/month on top of the base and compute costs. [Backup documentation](https://supabase.com/docs/guides/platform/backups), [pricing](https://supabase.com/pricing).

**Proposal:** accept daily backups only if losing approximately a day's committed data is acceptable. Define a recovery time target and run a restore drill before relying on the service. A second nightly export increases independence but does not inherently improve a daily recovery point. More frequent exports or PITR require a conscious budget decision. Offline replicas are not backups: devices may be lost, missing data or holding old authorization state. Retention and restoration must account for revoked memberships and deletion requests.

## Decision gates and later validation

1. Confirm the sign-in journeys: Apple, optional Google, and email link or OTP. Select production email delivery and test existing-account invitation acceptance.
2. Define who may invite, remove a member, transfer ownership and delete a household; resolve how bearer invitations are bound to their recipient.
3. Define what EU hosting means for storage, execution, logs and subprocessors, then choose Edge or dedicated API against that requirement.
4. Align the synchronization mechanism with the command endpoint, database role, membership revocation and idempotency protocol. Framework selection alone does not provide offline synchronization.
5. Establish acceptable data-loss and recovery-time targets, monthly budget and backup ownership.
6. When implementation is authorized, validate concurrent invitation acceptance, cross-household reads/writes, direct API bypass attempts, pooled identity isolation, stale-token revocation, duplicate retries with conflicting payloads, and restoration followed by sync.

No provider, host, role matrix or infrastructure is selected by this note.
