# Notifications

## Decided

The level of notifications included in V1 has not been finalized yet.

## To validate

Two categories are proposed:

- Local notifications for information already known: upcoming event, pick-up and check-in reminder.
- Server notifications for changes or facts recorded by another adult.

Example of expected behavior to be specified: when a pick-up is recorded and then synchronized, the second phone refreshes the day and removes the reminder that is no longer needed.

A silent push has been considered to facilitate synchronization. Actual recovery must also be defined when the application is opened and when the network returns; delivery and execution guarantees will be verified when the technical choice is made.

Expo Notifications and Expo Push are candidates, without a final selection. Preferences, delays, quiet hours and behavior after schedule changes remain to be decided.

The counter-analysis proposes generic external messages, with no first name, place or sensitive time. This replaces the initial idea of notifications detailing a child's pick-up, subject to validation of the privacy framing.

## Rejected/deferred

Rich notifications and advanced actions are proposed for after V1. A notification is not the source of truth for a check-in.
