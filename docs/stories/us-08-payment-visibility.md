# US-08 — Payment visibility (FR-05, NFR-01, NFR-02)

**Status:** Written; ready for learner/mentor review. Implementation not started.
**Sprint 1 delivery:** Specify this story and its acceptance criteria.

## User story

As a fitness-company administrator, I want to inspect payments so I can confirm whether a client has paid for a purchased session or assessment.

## Acceptance criteria

- **US-08-AC01:** Given requests in different states, list customer, original service/price snapshot, currency, creation time, and status; show an explicit empty state when none exist.
- **US-08-AC02:** Given a selected request, detail shows its ID, available Stripe session/payment reference, status, and paid time when confirmed.
- **US-08-AC03:** Given only an unverified browser return, show Pending rather than confirmed payment.
- **US-08-AC04:** Given later customer/service edits, preserve customer ID association and original service/price snapshot.
- **US-08-AC05:** Given the NFR-01 workload, list/detail meet the timing target. Reports/exports are not required.

## Dependencies

US-01 for access; US-04, US-06, and US-07 for payment records and state.

## Completion and review

- [x] User, need, and benefit are stated.
- [x] Acceptance criteria have identifiable, observable outcomes.
- [x] Requirement mapping and dependencies are documented.
- [ ] Learner/mentor feedback incorporated, if any.
- [ ] Feature implemented and acceptance criteria verified in a later sprint.

The unchecked implementation item does not block the Sprint 1 written-story deliverable.

[All eight stories](README.md) · [Full requirements](../requirements.md)
