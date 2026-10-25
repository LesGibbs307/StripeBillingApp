# One-page project brief

**Project:** RepRise | **Company:** RepRise Fitness (fictional)

**Phase:** Sprint 1, Weeks 1–2 — requirements and GitHub workflow

**Status:** Draft for learner critique and mentor review; no implementation

**Problem.** Fitness staff manually track clients, session prices, and payment confirmations. This risks inconsistent charges and uncertainty about whether a personal-training session or fitness assessment has been paid for. The app will connect each client and purchased service to a verified one-time payment record.

**Target users.** The company owner or front-desk administrator maintains client records and requests payment. Fitness clients pay through Stripe-hosted Checkout without app accounts. The learner and mentor review scope and repository history.

**MVP scope.** One company; admin access; customer records; fixed-price services; one service per payment request; one-time Stripe payments; verified webhooks; basic admin payment list/detail. Assumptions: USD, cards only, fictional data, test mode, manually shared links.

**Future/out of scope.** Recurring fitness memberships/subscriptions add recurring lifecycle complexity; invoices add document and receivables workflows; refunds add reversal controls and reconciliation; advanced reporting adds analysis beyond individual payment verification. Also defer scheduling, attendance tracking, workout plans, health records, tax calculation, discounts, multiple currencies, customer accounts, automated email, multiple companies, and production launch. Sprint 1 documents the MVP; features are implemented later.

**Functional requirements (5).**

1. **FR-01:** Authenticate administrators and restrict management actions to them.
2. **FR-02:** Create, view, and update customer names and emails.
3. **FR-03:** Maintain active/inactive services with positive fixed USD prices.
4. **FR-04:** Create one-time Checkout requests for existing customers and active services using trusted price snapshots and safe retries.
5. **FR-05:** Verify webhooks, apply payment updates without duplicates, and display payment status to admins.

**Nonfunctional requirements (3).** Proposed future acceptance targets:

- **NFR-01 — Performance:** Admin list/detail p95 response time at most 2 seconds with 10 concurrent users, 1,000 customers, and 10,000 payments; measure Stripe latency separately.
- **NFR-02 — Security:** All management endpoints deny unauthorized access; invalid webhook signatures cause no payment changes; credentials/card data never enter source, app storage, or logs.
- **NFR-03 — Availability/recovery:** Target 99.5% monthly app availability excluding declared maintenance, with Stripe outages reported separately. After a simulated 5-minute app outage, replayed valid events converge within 60 seconds of receipt without duplicates.

**Sprint 1 success.** Review requirements with the mentor; explain exclusions confidently; provide a public repo with README, ignore rules, protected main, and at least three merged PRs with meaningful conventional commits. History and PR descriptions must tell the project's story. GitHub evidence and review are pending.

**Fallback.** Prioritize requirements and a basic public repo with at least one PR before Sprint 2. Both must exist, even if thin; this is not the full three-PR pass.
