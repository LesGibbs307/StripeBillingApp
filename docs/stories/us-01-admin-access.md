# US-01 — Admin access (FR-01, NFR-02)

**Status:** Written; ready for learner/mentor review. Implementation not started.
**Sprint 1 delivery:** Specify this story and its acceptance criteria.

## User story

As an administrator, I want to sign in and out so only authorized staff can manage billing records.

## Acceptance criteria

- **US-01-AC01:** Given a provisioned admin with valid credentials, sign-in permits customer, service, and payment management.
- **US-01-AC02:** Given invalid credentials, sign-in fails without revealing whether an account exists.
- **US-01-AC03:** Given an anonymous/non-admin caller, every management endpoint returns 401/403 as appropriate, discloses no business data, and makes no changes.
- **US-01-AC04:** Given logout, the previous session can no longer access protected endpoints.

## Dependencies

None. Defines access rules used by the other admin stories.

## Completion and review

- [x] User, need, and benefit are stated.
- [x] Acceptance criteria have identifiable, observable outcomes.
- [x] Requirement mapping and dependencies are documented.
- [ ] Learner/mentor feedback incorporated, if any.
- [ ] Feature implemented and acceptance criteria verified in a later sprint.

The unchecked implementation item does not block the Sprint 1 written-story deliverable.

[All eight stories](README.md) · [Full requirements](../requirements.md)
