# US-06 — Verified confirmation (FR-05, NFR-02)

**Status:** Written; ready for learner/mentor review. Implementation not started.
**Sprint 1 delivery:** Specify this story and its acceptance criteria.

## User story

As an administrator, I want verified payment notifications so payment status is trustworthy.

## Acceptance criteria

- **US-06-AC01:** Given a valid endpoint signature, matching test environment, known request/session, matching amount/currency, and confirmed payment success, update to Paid with Stripe reference and paid time.
- **US-06-AC02:** Given a missing/invalid signature or modified raw body, return 4xx and change no payment record.
- **US-06-AC03:** Given a valid event with unknown request or mismatched environment/amount/currency, do not mark Paid; record a diagnostic without credentials, card data, or raw sensitive payloads.
- **US-06-AC04:** Given a valid unsupported event, acknowledge it without changing payment state.

## Dependencies

US-04 for request correlation; US-07 defines retry and ordering behavior.

## Completion and review

- [x] User, need, and benefit are stated.
- [x] Acceptance criteria have identifiable, observable outcomes.
- [x] Requirement mapping and dependencies are documented.
- [ ] Learner/mentor feedback incorporated, if any.
- [ ] Feature implemented and acceptance criteria verified in a later sprint.

The unchecked implementation item does not block the Sprint 1 written-story deliverable.

[All eight stories](README.md) · [Full requirements](../requirements.md)
