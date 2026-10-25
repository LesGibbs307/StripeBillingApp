# US-02 — Customer records (FR-02)

**Status:** Written; ready for learner/mentor review. Implementation not started.
**Sprint 1 delivery:** Specify this story and its acceptance criteria.

## User story

As a fitness-company administrator, I want to maintain client names and contact details so payments attach to the correct client.

## Acceptance criteria

- **US-02-AC01:** Given a nonblank name and valid email, saving creates one customer with a stable ID visible in list/detail.
- **US-02-AC02:** Given a blank name or malformed email, saving reports validation errors and creates no record.
- **US-02-AC03:** Given an existing customer, name/email edits preserve its ID and payment associations.
- **US-02-AC04:** Given two customers with the same email, they may have distinct IDs; payments use the selected ID, never email alone. This explicit MVP policy requires mentor review.

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
