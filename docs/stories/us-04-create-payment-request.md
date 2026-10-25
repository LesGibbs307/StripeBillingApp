# US-04 — Create payment request (FR-04, NFR-02)

**Status:** Written; ready for learner/mentor review. Implementation not started.
**Sprint 1 delivery:** Specify this story and its acceptance criteria.

## User story

As a fitness-company administrator, I want a checkout link for a client and a personal-training session or fitness assessment so I can request a one-time payment.

## Acceptance criteria

- **US-04-AC01:** Given an existing customer and active service, creation stores customer ID, service ID/name snapshot, amount, currency, creation time, and request ID, and returns a test-mode Checkout URL.
- **US-04-AC02:** Given a client-tampered price, creation rejects it or ignores it in favor of the trusted price; it never creates the cheaper request.
- **US-04-AC03:** Given an unknown customer or inactive service, creation fails before creating a Checkout session.
- **US-04-AC04:** Given repeated submission with the same request identity, including after a timeout, it resolves to the same logical request/session rather than another payable session. A new intentional purchase has a new identity.
- **US-04-AC05:** Given Stripe unavailability, show a retryable error, retain the request identity, and never report payment success.

## Dependencies

US-01, US-02, and US-03; supplies the request used by US-05 and US-06.

## Completion and review

- [x] User, need, and benefit are stated.
- [x] Acceptance criteria have identifiable, observable outcomes.
- [x] Requirement mapping and dependencies are documented.
- [ ] Learner/mentor feedback incorporated, if any.
- [ ] Feature implemented and acceptance criteria verified in a later sprint.

The unchecked implementation item does not block the Sprint 1 written-story deliverable.

[All eight stories](README.md) · [Full requirements](../requirements.md)
