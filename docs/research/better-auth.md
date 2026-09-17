# Better Auth: PostgreSQL, Expo and local testing

> Decision update: [ADR 0007](../adr/0007-backend-stack-and-package-boundaries.md) selects NestJS, Effect, Prisma 7 and Better Auth. Candidate wording below records the earlier comparison. See the [executed prototype](../14-backend-compatibility-prototype.md) for current compatibility evidence.

Checked September 17, 2026. Status: candidate assessment, not a library decision. No package installation, migration, integration test or device test was executed for this note.

## Fit with the preferred direction

Better Auth documents a Drizzle adapter supporting PostgreSQL through the `pg` provider. This lets Foylo use its own backend and database connection. The adapter also supports an explicit PostgreSQL schema namespace. These are documented capabilities, not evidence that a particular combination of package versions works in Foylo. [Drizzle adapter](https://better-auth.com/docs/adapters/drizzle).

The documented workflow generates the auth schema from configuration, then uses Drizzle Kit for migrations. Proposed ownership: Foylo reviews and versions the generated auth schema and migration files alongside its domain migrations. Library upgrades must produce reviewed changes; production startup must not silently regenerate the schema. Table naming, identifiers and relationships need a compatibility check before implementation. [Schema and migrations](https://better-auth.com/docs/adapters/drizzle#schema-generation--migration).

## Native iPhone sign-in

The official Expo integration supports native clients, a separately hosted backend, secure cookie storage through SecureStore, and provider ID-token sign-in. Apple is among the supported ID-token providers. This makes native Apple authentication followed by server-side token verification a plausible path for Foylo, without requiring a browser redirect for that exchange. [Expo integration](https://better-auth.com/docs/integrations/expo), [Apple ID-token sign-in](https://better-auth.com/docs/authentication/apple#sign-in-with-apple-with-id-token).

Expo requires the Apple capability for the app's bundle identifier, `ios.usesAppleSignIn`, and the `expo-apple-authentication` configuration plugin or equivalent native configuration. Native configuration changes require a rebuilt app. [Expo configuration](https://docs.expo.dev/versions/latest/sdk/apple-authentication/#configuration-in-app-config).

Better Auth must accept the native bundle identifier as the token audience; confusing it with a web Service ID causes audience validation failure. Its Apple guide describes Developer credentials and client-secret generation. The browser OAuth variant additionally requires a registered HTTPS return URL and cannot use an HTTP localhost callback. Keep that constraint distinct from the native ID-token exchange, which has no redirect. Determine the exact credentials needed by the selected native-only implementation before provisioning anything. [Apple configuration](https://better-auth.com/docs/authentication/apple).

Native credentials can omit name or email on subsequent authorizations. Foylo must preserve initial profile information and handle absent values on repeat sign-in. Expo Go identifiers can differ from standalone builds; simulator support is incomplete, and checking credential revocation requires a physical device. [Expo development and API notes](https://docs.expo.dev/versions/latest/sdk/apple-authentication/).

## Local testing

Better Auth provides test utilities for user factories, database persistence, authenticated headers/cookies and OTP capture. These helpers belong in a separate test configuration. OTP capture is passive: it does not prevent the configured sender from transmitting messages. [Test utilities](https://better-auth.com/docs/plugins/test-utils).

Email verification and password-reset delivery use application-supplied callbacks. Proposed local setup: direct those callbacks to an in-memory inbox for automated tests and a local mail catcher for interactive checks. This is an integration design, not a built-in mail server. Verification links and reset tokens must be exercised through the real auth endpoints rather than only creating sessions with privileged helpers. [Email and password](https://better-auth.com/docs/authentication/email-password).

Proposed validation uses disposable PostgreSQL with the actual Drizzle adapter and reviewed migrations. Cover sign-up, verification, duplicate accounts, failed login, session expiry/revocation and reset-token reuse. Household permissions remain Foylo-specific tests. Production email deliverability, Apple consent, native credentials and device behavior cannot be established by a local inbox or session fixture. No such acceptance is claimed here.

## Assessment

For the requested clean architecture, the proposed boundary keeps Better Auth and its HTTP handler in infrastructure. Application use cases receive a provider-independent authenticated identity and enforce household permissions themselves. Domain objects must not depend on Better Auth session or user types. This is a Foylo design recommendation, not a claim of native Effect integration.

The documentation supports continuing the Better Auth evaluation: it matches the own-backend direction and offers useful local integration facilities. Retaining it still requires selecting compatible versions and proving the native login and database paths. Authentication method, backend framework, hosting and final library choice remain open.

## Sessions and the Foylo API

Better Auth documents database-backed sessions, configurable expiry/refresh and explicit revocation. Its optional cookie cache can continue accepting a revoked session until the cache expires. Proposed Foylo baseline: validate database-backed sessions without accepting stale cookie-cache authority on protected business operations, then separately verify current FamilyMembership. Revoking household access is not the same operation as revoking an authentication session. [Session management](https://better-auth.com/docs/concepts/session-management).

The Expo plugin caches session information in SecureStore and handles cookies for auth calls. Custom native requests to the Foylo API must explicitly forward the session cookie. A cached mobile session supports local presentation but cannot prove current server authorization or implement the offline command queue. [Expo authenticated requests](https://better-auth.com/docs/integrations/expo#making-authenticated-requests-to-your-server).

The library exposes a standard Request/Response handler; the documented Hono integration mounts it directly and obtains sessions for business middleware. Hono remains a candidate, not a framework decision. Effect-based application use cases may call an authentication port whose adapter translates Better Auth results and failures; this is Promise interoperability, not a documented native Effect adapter. Do not expose Better Auth user/session types to business contracts. [Hono integration](https://better-auth.com/docs/integrations/hono).

Required later checks include a revoked session with cached client state, a revoked membership with a valid session, authenticated native calls to our own endpoints, and expired authentication while local commands await upload. No behavior was executed in this research pass.
