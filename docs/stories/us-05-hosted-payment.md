# US-05 — Hosted payment (FR-04)

**Status:** Written; ready for learner/mentor review. Implementation not started.
**Sprint 1 delivery:** Specify this story and its acceptance criteria.

## User story

As a fitness client, I want to pay for a personal-training session or fitness assessment through hosted Checkout without an app account.

## Acceptance criteria

- **US-05-AC01:** Given a valid link, Checkout shows the service, frozen amount, and USD currency without app login.
- **US-05-AC02:** Given successful test-card payment, the customer can return to a confirmation page; browser return alone cannot mark the app payment Paid.
- **US-05-AC03:** Given a declined card, the customer sees an error and the app does not mark Paid; a later successful retry can complete the purchase.
- **US-05-AC04:** Given cancellation or a closed browser, do not infer payment success. Modified return parameters cannot change payment state.
- **US-05-AC05:** Given only a checkout link, the customer cannot access customer management or other payment records.

## Dependencies

US-04 for the checkout link; US-06 for authoritative payment confirmation.

## Completion and review

- [x] User, need, and benefit are stated.
- [x] Acceptance criteria have identifiable, observable outcomes.
- [x] Requirement mapping and dependencies are documented.
- [ ] Learner/mentor feedback incorporated, if any.
- [ ] Feature implemented and acceptance criteria verified in a later sprint.

The unchecked implementation item does not block the Sprint 1 written-story deliverable.

[All eight stories](README.md) · [Full requirements](../requirements.md)
