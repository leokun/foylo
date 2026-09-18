# Platforms

## Decided

The chosen mobile foundation is Expo / React Native / TypeScript. Business logic must be shareable.

V1 delivery is iPhone-only, confirmed with the [V1 scope](01-v1-scope.md) on September 17, 2026. The product owner also confirmed a French-only user interface for V1.

## Initial household test devices

The product owner reported an iPhone 14 and an iPhone 14 Pro, running iOS 26 and iOS 27. The exact model-to-version pairing and OS builds have not been confirmed. These are the intended physical test devices, not evidence of successful installation or compatibility, and not yet the minimum supported OS decision.

## To validate

The first trial is limited to the product owner's household. Minimum versions, supported iPhone models and the technical distribution method remain to be chosen.

Android remains a future possibility, consistent with the mobile choice. A web interface could make it easier to configure weekly templates and browse history.

The Watch is envisioned as a lightweight interface for the next task and quick actions. The design direction is a SwiftUI application connected to the iPhone through WatchConnectivity. Feasibility, distribution constraints and behavior when the phone is absent remain to be verified before any commitment.

Preserving a logic oriented toward simple actions from the design stage is proposed for this evolution, without building a Watch client or locking in API routes today.

## Rejected/deferred

Android, web and Watch are outside V1 to avoid running several clients at once. Widgets, complications and Live Activities are not included in any delivery commitment.
