# Authentication and permissions

## Decided

No authentication provider or permissions matrix has been chosen yet.

## To validate

The household would be the primary access scope. The proposed model goes through FamilyMembership between User and Family, even though V1 usage is limited to one family per account and two adults. This avoids hard-coding a single membership directly in User.

A Person shown in the schedule does not necessarily have an account. A grandparent could be designated as a responsible adult without being granted access to the application.

The roles under consideration are OWNER, ADULT and MEMBER. Their exact rights remain to be defined: invitations, household management, schedule changes, check-in, correction and data deletion.

The proposed methods are Apple, possibly Google, and email with a sign-in link. Invitations by link or QR code, expiration, single use, access removal and recovery of the owner account remain to be specified.

Server-side access will have to verify active household membership for every operation. Handling data already present on a disconnected device after revocation must be addressed separately from cutting off server access.

## Rejected/deferred

Multi-household support, sharing a child between households and fine-grained permissions for external caregivers are proposed for after V1. A Grant table was suggested in the counter-analysis; creating it immediately has not been decided.
