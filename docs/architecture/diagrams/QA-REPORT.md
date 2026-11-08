# Mermaid diagram QA — 2026-11-08

Reviewed all 16 standalone diagrams against the MVP module design and the 36 acceptance criteria. Corrected the issues below, rendered every final source successfully to SVG and PNG, and visually inspected four review sheets.

Renderer: Mermaid 12.3.0, Mermaid CLI 12.2.0, headless Chromium 156.0.8078.47. Sequence-message semicolons caused a real parser failure and were replaced with commas. Structural checks alone in the earlier review had not detected it.

| Diagram | Syntax/render | Logic review and correction |
| --- | --- | --- |
| [01-system-overview.mmd](01-system-overview.mmd) | PASS | Moved browser execution outside the server process and the Billing Stripe adapter inside it. |
| [02-purchase-sequence.mmd](02-purchase-sequence.mmd) | PASS | Fixed parser-breaking semicolons; modeled browser return and webhook as independent paths; showed authorization. |
| [03-admin-login.mmd](03-admin-login.mmd) | PASS | Separated one-time provisioning from login and added Identity-store failure. |
| [04-authorization-and-logout.mmd](04-authorization-and-logout.mmd) | PASS | Added fail-closed session-store failure and separated read, mutation and logout paths. |
| [05-customer-write.mmd](05-customer-write.mmd) | PASS | Made row-version enforcement atomic at commit and distinguished conflicts from dependency failures. |
| [06-customer-read-and-association.mmd](06-customer-read-and-association.mmd) | PASS | Added invalid-input and database-failure outcomes; retained stable-ID association. |
| [07-service-management.mmd](07-service-management.mmd) | PASS | Added commit-time version conflict; retained history and deactivation behavior. |
| [08-service-price-snapshot.mmd](08-service-price-snapshot.mmd) | PASS | Added rollback/lock-release path when snapshot commit fails. |
| [09-checkout-creation.mmd](09-checkout-creation.mmd) | PASS | Prevented resume from bypassing safe recovery; added first-attempt persistence and binding/concurrency outcomes. |
| [10-checkout-recovery.mmd](10-checkout-recovery.mmd) | PASS | Added lease, never-attempted request, rejected request, safe-window, binding and persistence decisions. |
| [11-hosted-payment.mmd](11-hosted-payment.mmd) | PASS | Removed misleading Pending resets; kept browser return independent of verified state. |
| [12-webhook-processing.mmd](12-webhook-processing.mmd) | PASS | Added diagnostic commit failure, concurrent receipt recheck, session binding and duplicate-key rollback paths. |
| [13-payment-states.mmd](13-payment-states.mmd) | PASS | Made Expired no-op behavior explicit while preserving authoritative-success precedence. |
| [14-outage-recovery.mmd](14-outage-recovery.mmd) | PASS | Added signature/evidence verification and provider response; removed ambiguous semicolons. |
| [15-payment-queries.mmd](15-payment-queries.mmd) | PASS | Added invalid-query and dependency-failure outcomes. |
| [16-data-relationships.mmd](16-data-relationships.mmd) | PASS | Added account/environment components of event key and nullable relationship/reference annotations. |

## Verification boundaries
- All 16 final diagrams rendered successfully. Source, standalone previews and embedded architecture diagrams are synchronized.
- Requirements coverage remains 36 of 36; module ownership and MVP exclusions are unchanged.
- Rendered sheets showed no clipped labels or nodes. Long Billing workflows require scrolling or zoom; open the SVG for detailed review rather than a reduced thumbnail.
- The ER diagram is a core-field view; the complete persistence dictionary remains in [data-model.md](../data-model.md).
- Diagram QA does not prove application behavior. No implementation, security, load or outage tests were executed. No GitHub actions were taken.
- Source hashes and per-file results are recorded in [qa-results.json](qa-results.json).

## References
The review used [Mermaid usage documentation](https://mermaid.js.org/config/usage.html), [Stripe webhook delivery guidance](https://docs.stripe.com/webhooks) and [Stripe idempotency semantics](https://docs.stripe.com/api/idempotent_requests). Provider retries require a stable key; potentially expired keys must not be retried blindly. These rules inform the proposed design rather than demonstrating a working integration.
