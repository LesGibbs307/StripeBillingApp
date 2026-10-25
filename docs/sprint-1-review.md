# Sprint 1 review and evidence

**Current state:** Local documentation prepared. The public repository and merged framing/requirements work are verified. Requirements review, mentor sign-off, main protection verification, and the final workflow-stage merge remain pending. No product acceptance tests have run because there is no application.

The [eight individual stories](stories/README.md) are now written and ready for review. The local project is connected to GitHub, and the stories are on main. Mentor review follows story preparation and is not a blocker to writing the stories or preparing the PRs.

## Weeks 1–2

| Window | Work | Evidence |
| --- | --- | --- |
| Week 1 | Review problem/users/assumptions; framing PR 1; critique and refine requirements in PR 2. | Brief, requirements, real PR links, critique notes |
| Week 2 | Review with mentor; workflow/review PR 3; verify protection and history. | Mentor notes, public URL, three merged PR links, protection evidence |

## Baseline checklist

- [x] One-page brief drafted: problem, users, scope, 5 functional and 3 nonfunctional requirements.
- [x] Requirements drafted with 8 stories and testable acceptance criteria.
- [x] README, .gitignore, PR template, and recommended workflow prepared.
- [x] No application code or feature implementation added.
- [ ] Learner has critiqued assumptions and corrected the draft.
- [ ] Learner and mentor have reviewed requirements together.
- [ ] Learner explains exclusions and reasons without hesitation.
- [x] Public repository was verified readable while signed out.
- [ ] Main protection is configured and verified.
- [x] GitHub shows three merged PRs (#1, #2, #3), including requirements history; #2 targeted the framing branch and #3 integrated the work into main.
- [ ] Final workflow-stage PR reviewed and merged.
- [ ] Mentor can follow the project story from commits and PR descriptions alone.

## Evidence register

| Item | Actual evidence | Status |
| --- | --- | --- |
| Public repository | [Repository](https://github.com/LesGibbs307/StripeBillingApp); signed-out access verified during setup | Public |
| Main protection | Settings evidence and check date pending | Not configured/verified |
| Framing PR | [#1](https://github.com/LesGibbs307/StripeBillingApp/pull/1), commit 4750862 | Merged into main |
| Requirements PR | [#2](https://github.com/LesGibbs307/StripeBillingApp/pull/2), commit 89d8869; [#3](https://github.com/LesGibbs307/StripeBillingApp/pull/3), integration merge 74fa09b | #2 merged into framing; #3 merged into main |
| Workflow/review PR | Branch docs/sprint-one-review; actual PR URL/merge evidence to be recorded after publication | Prepared; merge pending |
| Mentor review | Date, reviewer, findings pending | Not conducted |
| History review | Requirements commit link and notes pending | Not conducted |

## Mentor prompts

1. What problem does this solve, and who uses it?
2. Why exclude recurring memberships, scheduling, invoices, refunds, and advanced reporting?
3. Why does a checkout return page not prove payment?
4. What happens to duplicate, delayed, or out-of-order events?
5. What happens to an existing request when a service price changes?
6. Which acceptance criteria describe future features, and which deliverables are due in Sprint 1?
7. Does repository history explain the project without this conversation?

## Fitness-company talking point

> This project helps a fitness company track whether clients have paid for individual personal-training sessions or assessments. I focused on one-time payments first because memberships introduce renewals and cancellations. Scheduling and workout tracking are separate problems. Verified Stripe webhooks establish payment status, even if the client closes the browser before returning to the app.

## Requirements review log

Record review findings and corrections below. Review and approval remain pending.

| Reviewer/date | Assumption or issue challenged | Correction and reason | PR/commit |
| --- | --- | --- | --- |
| Pending | USD/cards-only, duplicate-email policy, price snapshots, performance/availability targets | Pending requirements review | Pending |

## Fallback

If time is short, prioritize requirements and a basic public repo with at least one PR before Sprint 2. Both must exist, even if thin. Record missing baseline evidence explicitly. This fallback is not the full three-merged-PR pass; implementing features does not compensate for missing Sprint 1 work.
