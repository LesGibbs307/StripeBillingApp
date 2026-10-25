# US-07 — Retries and ordering (FR-05, NFR-03)

**Status:** Written; ready for learner/mentor review. Implementation not started.
**Sprint 1 delivery:** Specify this story and its acceptance criteria.

## User story

As an administrator, I want repeated/delayed events handled safely so records remain accurate.

## Acceptance criteria

- **US-07-AC01:** Given the same valid event twice, including concurrently, produce one business update and one payment record.
- **US-07-AC02:** Given different events describing the same successful payment, keep one Paid outcome without duplicate purchases.
- **US-07-AC03:** Given an old failure/expiry event after success, never downgrade Paid.
- **US-07-AC04:** Given persistence failure, do not acknowledge successful handling before the update is durable or durably queued; a later delivery can complete it.
- **US-07-AC05:** Given restoration after a 5-minute outage, replaying a missed success event twice produces one Paid result within 60 seconds of receipt.

## Dependencies

US-06 for signature verification and payment correlation.

## Completion and review

- [x] User, need, and benefit are stated.
- [x] Acceptance criteria have identifiable, observable outcomes.
- [x] Requirement mapping and dependencies are documented.
- [ ] Learner/mentor feedback incorporated, if any.
- [ ] Feature implemented and acceptance criteria verified in a later sprint.

The unchecked implementation item does not block the Sprint 1 written-story deliverable.

[All eight stories](README.md) · [Full requirements](../requirements.md)
