# Eight user stories — Sprint 1

All eight written stories are prepared for review. They specify the future MVP; no app features are implemented. GitHub connection and mentor review are not prerequisites for creating these documents.

The requirements document is the source of truth. Keep these review copies synchronized when changing acceptance criteria.

| Story | Requirement mapping | Depends on |
| --- | --- | --- |
| [US-01 — admin access](us-01-admin-access.md) | FR-01, NFR-02 | None. Defines access rules used by the other admin stories. |
| [US-02 — customer records](us-02-customer-records.md) | FR-02 | US-01 for admin authorization. |
| [US-03 — service prices](us-03-service-prices.md) | FR-03 | US-01 for admin authorization. |
| [US-04 — create payment request](us-04-create-payment-request.md) | FR-04, NFR-02 | US-01, US-02, and US-03; supplies the request used by US-05 and US-06. |
| [US-05 — hosted payment](us-05-hosted-payment.md) | FR-04 | US-04 for the checkout link; US-06 for authoritative payment confirmation. |
| [US-06 — verified confirmation](us-06-verified-confirmation.md) | FR-05, NFR-02 | US-04 for request correlation; US-07 defines retry and ordering behavior. |
| [US-07 — retries and ordering](us-07-retries-and-ordering.md) | FR-05, NFR-03 | US-06 for signature verification and payment correlation. |
| [US-08 — payment visibility](us-08-payment-visibility.md) | FR-05, NFR-01, NFR-02 | US-01 for access; US-04, US-06, and US-07 for payment records and state. |

## How these become three PRs

The three planned stages are project framing, requirements and stories, then contribution workflow and mentor-review preparation. Actual GitHub PR numbers differ because an additional integration PR was merged. The sprint requires three meaningful PRs, not one PR per story and not three implemented features.

See the [delivery record](../pull-request-plan.md) and [review checklist](../sprint-1-review.md).
