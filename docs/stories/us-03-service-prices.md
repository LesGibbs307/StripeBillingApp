# US-03 — Service prices (FR-03)

**Status:** Written; ready for learner/mentor review. Implementation not started.
**Sprint 1 delivery:** Specify this story and its acceptance criteria.

## User story

As a fitness-company administrator, I want fixed prices for personal-training sessions and fitness assessments so payment requests use agreed amounts.

## Acceptance criteria

- **US-03-AC01:** Given the service name "Personal-training session (60 minutes)" and an illustrative price of 125.00 USD, saving creates an active service displaying $125.00.
- **US-03-AC02:** Given an empty name, nonpositive amount, or more than two decimal places, saving fails without changing the record.
- **US-03-AC03:** Given deactivation, the service remains in history but cannot be used for a new request.
- **US-03-AC04:** Given a price edit, new requests use the new price; existing requests keep their original price snapshot.

## Dependencies

US-01 for admin authorization.

## Completion and review

- [x] User, need, and benefit are stated.
- [x] Acceptance criteria have identifiable, observable outcomes.
- [x] Requirement mapping and dependencies are documented.
- [ ] Learner/mentor feedback incorporated, if any.
- [ ] Feature implemented and acceptance criteria verified in a later sprint.

The unchecked implementation item does not block the Sprint 1 written-story deliverable.

[All eight stories](README.md) · [Full requirements](../requirements.md)
