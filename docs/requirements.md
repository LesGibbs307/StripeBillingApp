# RepRise — requirements

**Version:** 0.1 | **Status:** Ready for review
**Sprint:** 1, Weeks 1–2 — project definition, not feature implementation

## 1. Problem and users

RepRise is a Stripe payment app for RepRise Fitness, a fictional fitness company offering individual personal-training sessions and fitness assessments. Staff need a reliable relationship between clients, agreed service prices, and confirmed payments. The owner or front-desk administrator maintains records and requests payment; clients pay through hosted Checkout without an app account. In these requirements, customer means a fitness client. See the [brief](project-brief.md).

The eight stories define the future MVP backlog. Sprint 1 specifies and reviews them; product acceptance tests will be executed after implementation.

## 2. Scope and assumptions

| In the future MVP | Boundary |
| --- | --- |
| Admin access | Provisioned accounts, one admin role, login/logout; no public registration |
| Customer records | Stable ID, name, email; create/read/update; no deletion workflow |
| Service pricing | Name, positive fixed USD price, active flag; create/read/update/deactivate |
| One-time Stripe payments | Existing customer, one active service, quantity one; hosted Checkout; cards only |
| Verified webhooks | Authenticity checks, safe retry/duplicate handling, recorded payment state |
| Basic payment view | Individual payment list/detail; no analytics/exports |

Planning assumptions for mentor review: one company, USD, fictional data, test mode, manually shared links, no discounts or calculated tax. Prices represent final agreed demonstration amounts. Real accounting/tax and production decisions are deferred. Checkout confirmation does not imply invoice generation.

| Future/out of scope | Reason |
| --- | --- |
| Recurring memberships/subscriptions | Recurrence, renewals, cancellation, and prorations introduce another lifecycle |
| Invoices | Numbered documents, due dates, and receivables add a separate workflow |
| Refunds | Reversals need additional permissions, audit rules, and reconciliation |
| Advanced reporting | Aggregation, dashboards, and exports exceed individual payment verification |
| Tax, discounts, multiple currencies | Additional pricing rules distract from the core flow |
| Customer accounts, automatic email, multiple companies | Additional identity, delivery, and isolation concerns |
| Scheduling, attendance, workout plans, and health records | Separate fitness operations and sensitive data are unnecessary for confirming payment |
| Live payments and production deployment | Require a later operational readiness review |

### Fitness scenario for discussion

An administrator creates a client record, selects a 60-minute personal-training session priced at an illustrative $125, and shares a one-time checkout link. The client pays through Stripe; a verified webhook confirms payment in the admin view. A fitness assessment is another separately priced service, using the same flow. Prices are fictional examples, not market recommendations.

The app records payment, not appointment booking or completion. Scheduling remains outside the app. No injury history, health measurements, workout plans, or other health records are collected by this MVP.

## 3. Functional requirements (5)

| ID | Required behavior | Stories |
| --- | --- | --- |
| FR-01 | Authenticate provisioned admins, support logout, and authorize every management operation server-side. | US-01 |
| FR-02 | Create, view, and edit customers with required name and valid email. | US-02 |
| FR-03 | Create, view, edit, and deactivate services; accept positive fixed USD prices with at most two decimal places. | US-03 |
| FR-04 | Create one-time Checkout requests for existing customers and active services using trusted price snapshots and safe retries. | US-04, US-05 |
| FR-05 | Validate Stripe webhooks, correlate payments, update states safely, and expose admin payment list/detail. | US-06, US-07, US-08 |

## 4. Nonfunctional requirements (3)

Proposed product targets, not measured results or Sprint 1 guarantees.

| ID | Target | Future verification |
| --- | --- | --- |
| NFR-01 — Performance | Admin list/detail p95 at most 2 seconds with 10 concurrent users, 1,000 customers, and 10,000 payments. Measure Stripe latency separately. | Document staging configuration; warm up 1 minute, then measure a 5-minute mixed list/detail workload; report percentiles and errors. |
| NFR-02 — Security | Unauthorized callers cannot read/change admin data. Invalid signatures cannot change payments. No credentials/card data in source, app database, or logs. HTTPS outside local development. | Exercise all protected endpoints as anonymous/non-admin; test missing/invalid signatures; inspect source, test data, and logs. Use environment configuration or a secret store for credentials. |
| NFR-03 — Availability/recovery | Target 99.5% monthly app availability excluding declared maintenance; report Stripe outages separately. After a simulated 5-minute outage, replay converges within 60 seconds of receipt without duplicate records. | One-minute health probes: eligible successful probes / eligible total probes. Stop/restore app, replay a missed event twice, check status and row count. Monthly availability can only be measured after deployment. |

## 5. User stories and acceptance criteria (8)

### US-01 — Admin access (FR-01, NFR-02)

As an administrator, I want to sign in and out so only authorized staff can manage billing records.

- Given a provisioned admin with valid credentials, sign-in permits customer, service, and payment management.
- Given invalid credentials, sign-in fails without revealing whether an account exists.
- Given an anonymous/non-admin caller, every management endpoint returns 401/403 as appropriate, discloses no business data, and makes no changes.
- Given logout, the previous session can no longer access protected endpoints.

### US-02 — Customer records (FR-02)

As a fitness-company administrator, I want to maintain client names and contact details so payments attach to the correct client.

- Given a nonblank name and valid email, saving creates one customer with a stable ID visible in list/detail.
- Given a blank name or malformed email, saving reports validation errors and creates no record.
- Given an existing customer, name/email edits preserve its ID and payment associations.
- Given two customers with the same email, they may have distinct IDs; payments use the selected ID, never email alone. This explicit MVP policy requires mentor review.

### US-03 — Service prices (FR-03)

As a fitness-company administrator, I want fixed prices for personal-training sessions and fitness assessments so payment requests use agreed amounts.

- Given the service name "Personal-training session (60 minutes)" and an illustrative price of 125.00 USD, saving creates an active service displaying $125.00.
- Given an empty name, nonpositive amount, or more than two decimal places, saving fails without changing the record.
- Given deactivation, the service remains in history but cannot be used for a new request.
- Given a price edit, new requests use the new price; existing requests keep their original price snapshot.

### US-04 — Create payment request (FR-04, NFR-02)

As a fitness-company administrator, I want a checkout link for a client and a personal-training session or fitness assessment so I can request a one-time payment.

- Given an existing customer and active service, creation stores customer ID, service ID/name snapshot, amount, currency, creation time, and request ID, and returns a test-mode Checkout URL.
- Given a client-tampered price, creation rejects it or ignores it in favor of the trusted price; it never creates the cheaper request.
- Given an unknown customer or inactive service, creation fails before creating a Checkout session.
- Given repeated submission with the same request identity, including after a timeout, it resolves to the same logical request/session rather than another payable session. A new intentional purchase has a new identity.
- Given Stripe unavailability, show a retryable error, retain the request identity, and never report payment success.

### US-05 — Hosted payment (FR-04)

As a fitness client, I want to pay for a personal-training session or fitness assessment through hosted Checkout without an app account.

- Given a valid link, Checkout shows the service, frozen amount, and USD currency without app login.
- Given successful test-card payment, the customer can return to a confirmation page; browser return alone cannot mark the app payment Paid.
- Given a declined card, the customer sees an error and the app does not mark Paid; a later successful retry can complete the purchase.
- Given cancellation or a closed browser, do not infer payment success. Modified return parameters cannot change payment state.
- Given only a checkout link, the customer cannot access customer management or other payment records.

### US-06 — Verified confirmation (FR-05, NFR-02)

As an administrator, I want verified payment notifications so payment status is trustworthy.

- Given a valid endpoint signature, matching test environment, known request/session, matching amount/currency, and confirmed payment success, update to Paid with Stripe reference and paid time.
- Given a missing/invalid signature or modified raw body, return 4xx and change no payment record.
- Given a valid event with unknown request or mismatched environment/amount/currency, do not mark Paid; record a diagnostic without credentials, card data, or raw sensitive payloads.
- Given a valid unsupported event, acknowledge it without changing payment state.

### US-07 — Retries and ordering (FR-05, NFR-03)

As an administrator, I want repeated/delayed events handled safely so records remain accurate.

- Given the same valid event twice, including concurrently, produce one business update and one payment record.
- Given different events describing the same successful payment, keep one Paid outcome without duplicate purchases.
- Given an old failure/expiry event after success, never downgrade Paid.
- Given persistence failure, do not acknowledge successful handling before the update is durable or durably queued; a later delivery can complete it.
- Given restoration after a 5-minute outage, replaying a missed success event twice produces one Paid result within 60 seconds of receipt.

### US-08 — Payment visibility (FR-05, NFR-01, NFR-02)

As a fitness-company administrator, I want to inspect payments so I can confirm whether a client has paid for a purchased session or assessment.

- Given requests in different states, list customer, original service/price snapshot, currency, creation time, and status; show an explicit empty state when none exist.
- Given a selected request, detail shows its ID, available Stripe session/payment reference, status, and paid time when confirmed.
- Given only an unverified browser return, show Pending rather than confirmed payment.
- Given later customer/service edits, preserve customer ID association and original service/price snapshot.
- Given the NFR-01 workload, list/detail meet the timing target. Reports/exports are not required.

## 6. Shared rules and design boundaries

Minimum request states: Pending, Paid, Expired. A declined attempt leaves Pending while Checkout remains usable. Browser cancellation is not proof of expiry. Confirmed expiry may set Expired; authoritative verified success takes precedence if events arrive out of order. Paid is terminal within this MVP because reversals are outside scope.

Represent amounts in integer minor units conceptually (125.00 USD = 12,500 cents). Preserve the agreed amount on the request. Use event and provider payment/session identities to prevent duplicates and correlate updates. Exact event subscriptions, SDK versions, persistence, and authentication mechanism are later design decisions.

Stripe's [webhook documentation](https://docs.stripe.com/webhooks) informs signature/raw-body verification, duplicate handling, and ordering requirements. This is not a claim of an implemented integration.

## 7. Sprint 1 acceptance

The mentor and learner review this document together, the learner explains exclusions confidently, and a public repo demonstrates the files, protected main, and at least three merged PRs with conventional commits. Track evidence in the [review checklist](sprint-1-review.md) and [delivery record](pull-request-plan.md).

**Requirements review:** Review assumptions and acceptance criteria, and record any corrections and their reasons in the review log.

**Trend-based optional — scanning:** Consider CodeQL or Dependabot when eligible source/dependency files exist. Neither is enabled here. These optional extensions do not replace the baseline.
