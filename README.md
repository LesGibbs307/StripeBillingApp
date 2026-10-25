# RepRise — Sprint 1

RepRise is a Stripe payment app planned for **RepRise Fitness**, a fictional fitness company offering personal-training sessions and fitness assessments.

## Project framing

Read the [one-page project brief](docs/project-brief.md) for the problem, target users, MVP boundaries, five functional requirements, and three nonfunctional targets.

The future MVP includes admin access, client records, fixed service pricing, one-time Stripe-hosted payments, verified webhooks, and individual payment records. Recurring memberships, invoices, refunds, advanced reporting, scheduling, workout plans, and health records are outside this MVP.

Planning assumptions: one company, USD, card payments, fictional data, and Stripe test mode.

## Sprint 1 delivery

The project brief, README, ignore rules, and PR template establish the project framing. This change adds:

- [Detailed requirements](docs/requirements.md): five functional requirements and three measurable nonfunctional targets.
- [Eight user stories](docs/stories/README.md): individual review documents with 36 acceptance criteria and requirement/dependency mappings.

PR 3 will introduce the Git workflow, conventional-commit guidance, and mentor-review preparation.

## Current status

The local project is connected to [LesGibbs307/StripeBillingApp](https://github.com/LesGibbs307/StripeBillingApp). Branch protection and merged-PR evidence have not been verified. Requirements and mentor review remain pending.

No application code, dependencies, or payment integration exists. There are no build/run steps yet. The .NET solution is deferred until after requirements review, consistent with defining the project before implementation.

The full Sprint 1 pass requires a public repository, protected main, at least three merged PRs, and review with the mentor. Preparing files or making a local commit does not by itself satisfy those conditions.
