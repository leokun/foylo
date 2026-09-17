> Historical research preserved during branch consolidation. It contains earlier recommendations and unverified or superseded interpretations. Use the [consolidated note](../backend-and-auth.md) and current ADRs for decisions.

# Research: Supabase versus a dedicated API for auth, Postgres and business writes

Ticket: leokun/foylo#6 (part of #1). Research date: 2026-09-17. Sources are primary only (Supabase docs, pricing and legal pages; PostgreSQL docs; Hono, Fastify, Drizzle and Expo docs; Apple App Review Guidelines; Fly.io, Railway and Render docs). Most Supabase and framework doc pages carry no visible "last updated" date; where a date exists it is noted.

## Question

docs/03-architecture.md lists Supabase (auth plus managed Postgres) and Hono or Fastify as candidates. Compare: auth flows for two adults sharing a household, row-level security as a second line of defence, hosting business writes as Edge Functions or Postgres functions versus a dedicated API, EU data residency, backups, and cost at family scale.

## Summary

- Supabase Auth covers every flow the household needs on Expo: email/password, magic link or 6-digit email OTP (`signInWithOtp`), native Sign in with Apple and Google via `signInWithIdToken`, and invitation of the second adult via `auth.admin.inviteUserByEmail` or `generateLink` (server side, service role key). Apple's guideline 4.8 makes Sign in with Apple mandatory only if another third-party login (Google) is offered; guideline 5.1.1(v) requires in-app account deletion regardless.
- RLS on Supabase is a real second line of defence: `auth.uid()` policies over a `family_membership` table, with a `security definer` helper to avoid recursive policies, default deny, and the `service_role` bypass reserved to the server. Postgres itself warns that RLS does not stop covert channels via constraints, so business rules still belong in code.
- Business writes: Edge Functions (Deno, 256 MB, 2 s CPU, 150/400 s wall clock, pooled transaction-mode connections, `prepare: false`) fit short idempotent writes and can host Hono directly; Postgres functions give transactional `ON CONFLICT` upserts closest to the data; a dedicated Hono or Fastify API on Node gives a persistent connection, full validation stack and no runtime limits at the price of one more deployable. Drizzle works in all three (postgres-js driver, `drizzle-orm/supabase` RLS helpers, drizzle-kit migrations).
- EU residency: Supabase projects can be created in Paris, Frankfurt, Ireland, London, Zurich or Stockholm; Auth data lives in the project's `auth` schema in that region; the DPA (v1, effective 2026-08-01) commits to storing and primarily processing data in the chosen region and incorporates SCCs. Edge Functions run closest to the caller unless pinned with `x-region`.
- Backups: none on Free (manual `db dump` only), 7-day daily backups on Pro, PITR is a $100/month add-on that also needs Small compute: disproportionate at family scale.
- Cost at a few households: Supabase Free is $0 but pauses projects with low 7-day activity and its default SMTP sends 2 emails/hour to team addresses only; Pro is $25/month including one Micro instance. A dedicated Node API in the EU adds about $2 to $6/month (Fly.io Amsterdam shared-cpu-1x) or $5/month (Railway Hobby, Amsterdam).

| Option | Business writes | Validation and idempotency | Drizzle fit | EU residency | Backups | Monthly cost (family scale) |
|---|---|---|---|---|---|---|
| A. Supabase only: RLS + Postgres functions + Edge Functions | Edge Functions (Deno) and `rpc()` functions | Zod or Hono validator inside the function; `ON CONFLICT` in SQL; 2 s CPU and 256 MB limits | Yes via postgres-js with `prepare: false`, per-function `deno.json` | Project region in EU; pin functions with `x-region` | Free: none. Pro: 7 days daily. PITR add-on $100+ | Free $0 (pausing risk) or Pro $25 |
| B. Supabase (auth + Postgres) + dedicated Hono or Fastify API on Node | One API process, persistent Postgres connection | Full stack: Hono validator/Zod or Fastify Ajv/TypeBox; idempotency keys in a table plus `ON CONFLICT`; no runtime limits | Yes, direct connection or session pooler; drizzle-kit migrations | Supabase EU region + API in Fly ams/cdg/fra, Railway Amsterdam or Render Frankfurt | Same as A | A + $2 to $6 (Fly) or $5 (Railway) |
| C. Postgres functions only (no HTTP layer) | `security invoker` PL/pgSQL functions called via `rpc()` | Transactional, atomic upserts; validation in SQL only | Drizzle for schema and migrations; call functions via `sql` | Same as A | Same as A | Same as A |

Recommendation for the decision ticket: start with Supabase Auth plus Supabase Postgres in an EU region with RLS everywhere (options A or B share that base). Put business writes behind one HTTP entry point using Hono so the code can run first as an Edge Function (A) and move unchanged to a Node host (B) if the 2 s CPU limit, connection pooling or cold starts become a problem. Budget Pro ($25/month) before onboarding a second household, mainly for daily backups and leaked-password protection; skip PITR.

## Findings

### 1. Supabase Auth flows for two adults in one household

- Email and password: `signUp()` with optional email confirmation; password reset via `resetPasswordForEmail()` then `updateUser()`. Email auth is enabled by default. Source: https://supabase.com/docs/guides/auth/passwords
- Password policy: configurable minimum length (at least 8) and required character classes. Leaked password protection (HaveIBeenPwned Pwned Passwords API) is "available exclusively on the Pro Plan and above". Source: https://supabase.com/docs/guides/auth/password-security
- Magic link and email OTP share `signInWithOtp()`; the OTP variant only requires adding `{{ .Token }}` to the email template. Default: one request per 60 seconds per user, links and codes expire after 1 hour; this expiry setting is shared by confirmation, recovery, email change and invitation links. Source: https://supabase.com/docs/guides/auth/auth-email-passwordless
- Mobile deep links: on Expo, add the app scheme (`"scheme"` in app config) to the redirect URL allowlist (for example `com.supabase://**`), pass `emailRedirectTo: makeRedirectUri()` and turn the incoming URL into a session with `setSession()`. The docs recommend universal links for the best experience. Source: https://supabase.com/docs/guides/auth/native-mobile-deep-linking
- Sign in with Apple: native flow on iOS via AuthenticationServices / Expo, then `signInWithIdToken`; OAuth web flow also exists. Requirements: Team ID, App ID, Services ID, `.p8` signing key; "Apple requires you to generate a new secret key every 6 months" (web flow), and the user's full name is only provided on the first sign-in. Source: https://supabase.com/docs/guides/auth/social-login/auth-apple
- `expo-apple-authentication` supports iOS and tvOS only, "does not yet support Android or web", needs `ios.usesAppleSignIn: true` and returns an `identityToken` JWT verifiable against https://appleid.apple.com/auth/keys. Source: https://docs.expo.dev/versions/latest/sdk/apple-authentication/
- Google native sign-in on Expo: obtain an ID token with `@react-native-google-signin/google-signin`, pass it to `signInWithIdToken`; separate web, iOS and Android client IDs are required; nonce check is on by default. Source: https://supabase.com/docs/guides/auth/social-login/auth-google
- App Store rule: guideline 4.8 (Login Services) requires an equivalent privacy-preserving login (Sign in with Apple qualifies) when the app uses "a third-party or social login service (such as Facebook Login, Google Sign-In, ...)" for the primary account. It is not required if "Your app exclusively uses your company's own account setup and sign-in systems" (email/password or magic link only). Guideline 5.1.1(v): "If your app supports account creation, you must also offer account deletion within the app." Source: https://developer.apple.com/app-store/review/guidelines/ (no visible last-updated date)
- Inviting the second adult: `auth.admin.inviteUserByEmail(email, { redirectTo, data })` sends an invite link; `auth.admin.generateLink({ type: 'invite' | 'magiclink' | 'signup' | 'recovery' | 'email_change' })` returns the link and OTP without sending mail so the app can deliver it through its own channel (QR code, message). Both require the service role key and must run server side. Sources: https://supabase.com/docs/reference/javascript/auth-admin-inviteuserbyemail and https://supabase.com/docs/reference/javascript/auth-admin-generatelink
- Household invitation semantics (expiry, single use, revocation) are not a Supabase feature: they must be modelled in an app table (invitation token, family id, expires_at, accepted_by) and consumed by a server write that inserts the `family_membership` row.
- Email delivery limits: the default SMTP sends only to project team member addresses, "2 messages per hour", no SLA; custom SMTP (Resend, SES, Postmark, Brevo, etc.) is required for production and raises the initial limit to 30 messages per hour, adjustable. Sources: https://supabase.com/docs/guides/auth/auth-smtp and https://supabase.com/docs/guides/auth/rate-limits
- Sessions: access JWT expires after 1 hour by default; refresh tokens are single use with a 10-second reuse window; sign-out deletes the session but the JWT stays valid until expiry; time-boxed sessions, inactivity timeout and single session per user are "only available on Pro Plans and up". Revocation of a removed household member is therefore effective server side within at most one hour unless the API checks `auth.sessions` or the membership table on each write (which it should do anyway, see docs/05-auth-permissions.md). Source: https://supabase.com/docs/guides/auth/sessions
- Auth data location: user data lives in the `auth` schema of the project's Postgres database, and the Auth server (GoTrue) is deployed alongside the database in the project region. Source: https://supabase.com/docs/guides/auth/architecture
- Expo quickstart uses `@react-native-async-storage/async-storage` for session persistence. Source: https://supabase.com/docs/guides/auth/quickstarts/react-native

### 2. Row-level security as a second line of defence with a membership table

- PostgreSQL (docs for version 18): `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` switches the table to default deny; `USING` filters visible rows, `WITH CHECK` constrains inserted or updated rows; permissive policies are OR-ed, restrictive policies AND-ed; superusers, `BYPASSRLS` roles and table owners bypass RLS (owners unless `FORCE ROW LEVEL SECURITY`). Referential integrity checks bypass RLS, and the docs warn that policies do not protect against covert channels through constraint violations. Source: https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- Supabase pattern: enable RLS on every exposed table, revoke default grants for `anon` and `authenticated` and grant back only what is needed ("Adding policies doesn't take those grants back"); write one policy per command using `(select auth.uid()) = user_id`; for membership tables, break recursive policies with a `security definer` function in a private schema (for example `private.user_family_ids()` returning `setof uuid`); index the columns used in policies; the `service_role` key carries `bypassrls` and must never reach clients; test policies with `supabase test db`. Source: https://supabase.com/docs/guides/database/postgres/row-level-security
- Fit with Foylo: a `family_membership(user_id, family_id, role, status)` table plus policies of the form `family_id in (select private.user_family_ids())` on every family-scoped table gives the complementary defence described in docs/03-architecture.md. Role checks (OWNER, ADULT, MEMBER) can be expressed in policies too, but the docs' own recommendation for API-style logic is to keep RLS simple and put rules in server code.
- Production checklist: "Ensure you have enabled row level security (RLS) on all tables". Source: https://supabase.com/docs/guides/deployment/going-into-prod

### 3. Where business writes can live

Supabase Edge Functions
- Deno-compatible, TypeScript-first runtime, "globally distributed", "cold starts are possible", designed for "short-lived, idempotent operations"; Postgres should be treated as "a remote, pooled service". Source: https://supabase.com/docs/guides/functions
- Limits: 256 MB memory, 2 s CPU time per request (async I/O excluded), wall clock 150 s on Free and 400 s on paid plans, 150 s idle timeout, bundle 20 MB (CLI) / 5 MB (server bundling), 100 functions on Free, 1000 on Pro. Source: https://supabase.com/docs/guides/functions/limits
- Auth: `verify_jwt` defaults to true so the platform validates the JWT before the handler; the `withSupabase` wrapper exposes `ctx.userClaims` and an RLS-scoped `ctx.supabase` client plus `ctx.supabaseAdmin`. Source: https://supabase.com/docs/guides/functions/auth
- Postgres access: supabase-js, any Postgres driver, or Drizzle with postgres-js declared in the function's `deno.json`. Source: https://supabase.com/docs/guides/functions/connect-to-postgres
- Regional invocation: functions run in "the region closest to the user making the request" by default; pin with the `x-region` header or the client `region` option (`eu-west-3`, `eu-central-1`, `eu-west-1`, `eu-west-2`, `eu-central-2` available); pinning disables automatic failover. Source: https://supabase.com/docs/guides/functions/regional-invocation
- Hono runs inside Edge Functions: `import { Hono } from 'jsr:@hono/hono'` with `app.basePath('/<function-name>')`. Source: https://hono.dev/docs/getting-started/supabase-functions
- Quota: 500k invocations on Free, 2M on Pro/Team, $2 per additional million. Source: https://supabase.com/docs/guides/platform/manage-your-usage/edge-function-invocations
- No official idempotency or retry guidance beyond the "idempotent operations" advice; the development tips page only covers error types. Source: https://supabase.com/docs/guides/functions/development-tips

Postgres functions
- Created in SQL or PL/pgSQL, called with `rpc()` from every client library; `security invoker` is the default and the recommended choice; `security definer` functions must set `search_path` explicitly; `raise exception` aborts and rolls back the transaction. Source: https://supabase.com/docs/guides/database/functions
- Idempotent writes: `INSERT ... ON CONFLICT (...) DO NOTHING | DO UPDATE` requires a unique index or constraint as conflict target and "guarantees an atomic INSERT or UPDATE outcome ... even under high concurrency". This is the natural primitive for append-only check-ins keyed by a client-generated id. Source: https://www.postgresql.org/docs/current/sql-insert.html

Dedicated Hono or Fastify API on Node
- Hono: "Works on Cloudflare Workers, Fastly Compute, Deno, Bun, Vercel, Netlify, AWS Lambda, Lambda@Edge, and Node.js", first-class TypeScript, RPC mode for a typed client. Source: https://hono.dev/docs/
- Hono validation: thin built-in `validator` (json, form, query, header, param, cookie), `@hono/zod-validator`, `@hono/standard-validator` for Zod, Valibot and ArkType; validated data typed through `c.req.valid()`. Source: https://hono.dev/docs/guides/validation
- Hono on Node: `@hono/node-server`, Node 18.14.1 or later, `serve({ fetch: app.fetch, port })`, graceful shutdown handled manually. Source: https://hono.dev/docs/getting-started/nodejs
- Fastify (docs v5.12.5): Ajv v8 JSON Schema validation of body, query, params and headers, `fast-json-stringify` serialization, type providers and `setValidatorCompiler` for Zod, TypeBox, Joi or Yup; defaults `coerceTypes: 'array'`, `useDefaults: true`, `removeAdditional: true`. Source: https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/
- Fastify LTS: v5 released 2024-09-17, supports Node 20 and 22; v4 LTS ended 2025-06-30. Source: https://fastify.dev/docs/latest/Reference/LTS/
- Neither Hono nor Fastify core documents an idempotency-key middleware; idempotency has to be implemented with a unique key column and `ON CONFLICT`, in either the API or a Postgres function.
- Connection strategy for a persistent Node API: direct connection (IPv6 by default, IPv4 add-on) or Supavisor session mode on port 5432; transaction mode (port 6543) is for serverless and "does not support prepared statements"; a dedicated pooler exists on paid plans. Source: https://supabase.com/docs/guides/database/connecting-to-postgres

### 4. Drizzle with each option

- Drizzle connects to Supabase with the postgres-js driver; use the pooler URL with `prepare: false` for serverless, the direct connection for persistent servers. Source: https://orm.drizzle.team/docs/connect-supabase
- Drizzle can declare RLS: `pgTable.withRLS()`, `pgPolicy()` with `as`, `to`, `for`, `using`, `withCheck`; `drizzle-orm/supabase` exports `authenticatedRole`, `anonRole`, `serviceRole` and helpers; `drizzle.config.ts` `entities.roles` with `provider: 'supabase'` lets drizzle-kit manage policies without touching Supabase-owned roles; a `createDrizzle()` example runs queries in a transaction that sets the JWT claims and role via `set_config()` so RLS applies from the API as well. Source: https://orm.drizzle.team/docs/rls
- Drizzle in Edge Functions: per-function `deno.json` mapping `drizzle-orm/` and `postgres` to npm, `DATABASE_URL` from secrets, `prepare: false`; migrations either with `drizzle-kit generate` + `drizzle-kit migrate` or `drizzle-kit generate` + `supabase migration up`. Source: https://orm.drizzle.team/docs/tutorials/drizzle-with-supabase-edge-functions
- Consequence: the Drizzle schema (already decided in docs/03-architecture.md) can be the single source for tables, policies and migrations regardless of where the write logic runs.

### 5. EU region availability and data residency

- Supabase regions in Europe: Ireland `eu-west-1`, London `eu-west-2`, Paris `eu-west-3`, Frankfurt `eu-central-1`, Zurich `eu-central-2`, Stockholm `eu-north-1`. The selected region determines where "your primary project data is stored", and "Region selection is a data-location control, not proof of regulatory compliance." The page does not say whether the region can be changed later. Source: https://supabase.com/docs/guides/platform/regions
- DPA (version 1, effective 2026-08-01) is part of the standard terms; accepting the agreement counts as signing the SCCs; "Covered Data is stored and primarily Processed in that region unless otherwise required to comply with Customer's additional instructions"; sub-processor list published at https://supabase.com/legal/customer-resources/subprocessor-list with 30-day change notice. Source: https://supabase.com/legal/dpa
- Auth data is stored in the project database, so it follows the project region. Source: https://supabase.com/docs/guides/auth/architecture
- Edge Functions execute near the caller by default; for strict EU processing, pin `x-region` to the database region. Source: https://supabase.com/docs/guides/functions/regional-invocation
- Hosting for a dedicated API in the EU: Fly.io `ams` (Amsterdam), `cdg` (Paris), `fra` (Frankfurt), `lhr` (London), `arn` (Stockholm) (https://fly.io/docs/reference/regions/); Railway "EU West Metal" Amsterdam `europe-west4-drams3a` (https://docs.railway.com/reference/regions); Render Frankfurt, region fixed at creation (https://render.com/docs/regions).

### 6. Backups and point-in-time recovery per plan

- Daily backups: none on Free (export with `supabase db dump`), 7 days on Pro, 14 days on Team, 30 days on Enterprise. PITR: add-on on paid plans, needs at least Small compute, second-level granularity with a worst case RPO of 2 minutes, $0.137/hour (about $100/month) for 7 days, about $200 for 14 and $400 for 28 days; enabling PITR stops daily backups. Storage objects are not part of database backups. Source: https://supabase.com/docs/guides/platform/backups
- Pricing page confirms "Daily Backups Retention: None / 7 days / 14 days" and PITR "$100 per month per 7 days retention". Source: https://supabase.com/pricing
- Free plan projects can be paused for low activity over 7 days, with a 1-year one-click restore window; after that only a downloadable backup remains. Sources: https://supabase.com/docs/guides/deployment/going-into-prod and https://supabase.com/docs/guides/platform/upgrading

### 7. Realistic cost at family scale (a few households)

Supabase (https://supabase.com/pricing, https://supabase.com/docs/guides/platform/manage-your-usage/compute)
- Free: $0, 2 active projects, 500 MB database, 50k MAU, 5 GB egress, 500k function invocations, no backups, pausing after low activity, default SMTP only (2 emails/hour to team addresses): acceptable for a prototype, not for a second household.
- Pro: $25/month per organisation, includes $10 compute credits covering one Micro instance ($0.01344/hour, about $10/month), 8 GB database, 100k MAU, 250 GB egress, 2M invocations, 7-day daily backups, leaked-password protection and session controls. A second project costs about $10/month more (Micro). Paused projects are not billed.
- PITR: about $100/month plus Small compute ($15/month, minus $10 credits) so roughly $130/month total: out of proportion for a few households.
- Team: $599/month, irrelevant at this scale.

Dedicated Node API hosting (EU)
- Fly.io, usage based, Amsterdam prices: shared-cpu-1x 256 MB $2.02/month, 512 MB $3.32, 1 GB $5.92; stopped machines pay $0.15 per GB of rootfs per 30 days; no minimum. Source: https://fly.io/docs/about/pricing/
- Railway: Hobby $5/month with $5 usage included, Pro $20/month; $10 per GB RAM per month, $20 per vCPU per month, $0.05 per GB egress. Source: https://docs.railway.com/reference/pricing/plans
- Render: Free web services spin down after 15 minutes idle and share 750 instance hours per workspace per month, no persistent disk. Source: https://render.com/docs/free
- Order of magnitude: option A on Pro is $25/month; option B is $27 to $31/month; on Free tiers both are $0 to $6/month with the pausing and email caveats above.

## Open questions for the decision ticket

1. Do we ship Google sign-in in V1? If yes, Sign in with Apple becomes mandatory (guideline 4.8) and the 6-month Apple secret rotation must be owned by someone. If no, email plus magic link or OTP avoids both obligations.
2. Magic link (deep link, universal links setup) or 6-digit email OTP for the mobile flow? OTP avoids deep-link fragility on Expo; magic link is fewer taps.
3. Which custom SMTP provider, and from which domain? Required before any non-team user signs in.
4. Edge Functions first (option A) or Node API from day one (option B)? Proposed criterion: start A with Hono, move to B if CPU time, cold starts or connection handling hurt the offline sync endpoint.
5. Who owns the idempotency key for check-ins: client-generated UUID with `ON CONFLICT DO NOTHING`, or a dedicated `idempotency_keys` table storing the response for replay?
6. Is a 7-day daily backup on Pro acceptable, given PITR costs about $100/month? Should we add a nightly `pg_dump` to EU object storage as a cheap complement?
7. Which EU region: Paris (`eu-west-3`) for latency, or Frankfurt/Ireland for broader service availability? Confirm whether the region can be changed later (not documented).
8. Should RLS policies encode roles (OWNER, ADULT, MEMBER) or only membership, leaving role rules to server code?
9. Account deletion (guideline 5.1.1(v)) versus append-only history: what is deleted, anonymised or retained when an adult leaves or the household closes? Links to docs/08-security-privacy.md questions 5 and 6.

## Sources

Supabase
- https://supabase.com/pricing
- https://supabase.com/docs/guides/platform/backups
- https://supabase.com/docs/guides/platform/regions
- https://supabase.com/docs/guides/platform/manage-your-usage/compute
- https://supabase.com/docs/guides/platform/manage-your-usage/edge-function-invocations
- https://supabase.com/docs/guides/platform/upgrading
- https://supabase.com/docs/guides/deployment/going-into-prod
- https://supabase.com/legal/dpa (version 1, effective 2026-08-01)
- https://supabase.com/docs/guides/auth/passwords
- https://supabase.com/docs/guides/auth/password-security
- https://supabase.com/docs/guides/auth/auth-email-passwordless
- https://supabase.com/docs/guides/auth/native-mobile-deep-linking
- https://supabase.com/docs/guides/auth/social-login/auth-apple
- https://supabase.com/docs/guides/auth/social-login/auth-google
- https://supabase.com/docs/guides/auth/quickstarts/react-native
- https://supabase.com/docs/guides/auth/rate-limits
- https://supabase.com/docs/guides/auth/auth-smtp
- https://supabase.com/docs/guides/auth/sessions
- https://supabase.com/docs/guides/auth/architecture
- https://supabase.com/docs/reference/javascript/auth-admin-inviteuserbyemail
- https://supabase.com/docs/reference/javascript/auth-admin-generatelink
- https://supabase.com/docs/guides/database/postgres/row-level-security
- https://supabase.com/docs/guides/database/functions
- https://supabase.com/docs/guides/database/connecting-to-postgres
- https://supabase.com/docs/guides/functions
- https://supabase.com/docs/guides/functions/limits
- https://supabase.com/docs/guides/functions/auth
- https://supabase.com/docs/guides/functions/connect-to-postgres
- https://supabase.com/docs/guides/functions/regional-invocation
- https://supabase.com/docs/guides/functions/development-tips

PostgreSQL (documentation for version 18)
- https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- https://www.postgresql.org/docs/current/sql-insert.html

Frameworks and ORM
- https://hono.dev/docs/
- https://hono.dev/docs/guides/validation
- https://hono.dev/docs/getting-started/nodejs
- https://hono.dev/docs/getting-started/supabase-functions
- https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/ (Fastify v5.12.5)
- https://fastify.dev/docs/latest/Reference/LTS/
- https://orm.drizzle.team/docs/connect-supabase
- https://orm.drizzle.team/docs/rls
- https://orm.drizzle.team/docs/tutorials/drizzle-with-supabase-edge-functions

Apple and Expo
- https://developer.apple.com/app-store/review/guidelines/ (guidelines 4.8 and 5.1.1(v))
- https://docs.expo.dev/versions/latest/sdk/apple-authentication/

Hosting providers
- https://fly.io/docs/reference/regions/
- https://fly.io/docs/about/pricing/
- https://docs.railway.com/reference/regions
- https://docs.railway.com/reference/pricing/plans
- https://render.com/docs/regions
- https://render.com/docs/free
