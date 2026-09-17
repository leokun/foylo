# 0002: technical foundation

Date: September 17, 2026. Status: Decided.

## Context and decision

The final synthesis retains Expo / React Native with TypeScript, PostgreSQL and Drizzle. The domain is relational and sharing the TypeScript business logic is a goal.

## Alternatives and consequences

A fully native iPhone application was discussed to ease a future Watch app. The retained choice favors sharing the domain and the possibility of Android later on.

This decision selects neither an authentication provider, nor a hosting provider, nor an API framework, nor a synchronization tool. It does not trigger the creation of any code.
