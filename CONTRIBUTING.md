# Contributing to RepRise

## Repository and current delivery

The project is connected to [LesGibbs307/StripeBillingApp](https://github.com/LesGibbs307/StripeBillingApp). The initial repository, project framing, requirements, and eight stories already exist. Do not initialize another repository or repeat the bootstrap.

The final Sprint 1 documentation change adds this workflow, the [delivery record](docs/pull-request-plan.md), and the [mentor-review checklist](docs/sprint-1-review.md). Feature implementation remains outside Sprint 1.

## Branches and pull requests

Start each independent change from the current main branch:

```sh
git checkout main
git pull --ff-only origin main
git checkout -b docs/describe-the-change
# Edit the intended files, then stage explicit paths.
git add docs/requirements.md
git diff --cached
git diff --cached --check
git commit -m "docs: clarify payment acceptance criteria"
git push -u origin docs/describe-the-change
```

Use an appropriate new branch name and stage only files belonging to that change. Keep scratch files and screenshots out of commits. Do not use an indiscriminate add-all command when later work is still untracked.

Open a PR targeting main, use the repository's PR template, explain the change, record actual validation, inspect the diff, and resolve feedback before merging. Prefer merge commits to preserve the project story. Do not push directly to main or bypass branch protection.

If work must begin before its prerequisite PR merges, branch from that prerequisite and temporarily target its branch. Clearly document the dependency. Merge the prerequisite first, then retarget the dependent PR to main before merging it. GitHub PR numbers are not the same as delivery-stage numbers.

## Protect main

Protection has not been verified by this documentation change. Configure and verify a main-branch protection rule or equivalent ruleset:

- Require a pull request before merging and conversation resolution.
- Block force pushes and branch deletion.
- Enforce the rule for administrators/no bypass where supported.
- For solo work, leave required approving reviews disabled/zero and record self-review in the PR. If the mentor will review every PR, require one approval instead.
- Require status checks only after a real workflow exists and has run. This documentation-only project does not yet have CI.

Authors cannot approve their own PRs. See [GitHub review requirements](https://docs.github.com/en/pull-requests/how-tos/review-pull-requests/approving-a-pull-request-with-required-reviews) and [branch protection](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches).

## Conventional commits

Use `type: short imperative description`, optionally `type(scope): description`.

- `docs: clarify webhook acceptance criteria` — documentation.
- `chore: update repository ignore rules` — housekeeping.
- `feat: add customer records` — a future implemented feature.
- `fix: prevent duplicate payment updates` — a future behavior correction.

Keep commits focused and meaningful. Avoid vague messages or empty commits intended to inflate PR counts.

## Documentation checks

Before opening a documentation PR:

- Check scope against the one-page brief.
- Keep eight story statements and their numbered criteria synchronized with requirements.
- Verify relative links resolve to files included in the branch.
- Check the staged diff for whitespace errors, unintended files, credentials, and real customer data.
- Distinguish actual GitHub evidence from pending review, protection, or implementation work.

No application tests apply until application code exists. Proposed performance and recovery targets are not measured results.

## Review and completion

Mentor review follows preparation of the written stories. Record findings and subsequent corrections through PRs. A public repo and merged changes do not alone establish mentor approval or verified branch protection.

**Trend-based optional:** Consider CodeQL or Dependabot once eligible source or dependency files exist. Neither is configured by this change.
