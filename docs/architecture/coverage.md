# MVP coverage and verification
[Architecture index](README.md)

**Status:** Design traceability only. Checks below are planned implementation tests, not passing results. Acceptance criterion IDs refer to the existing [story documents](../stories/README.md).

## All 36 acceptance criteria
| Criterion | Workflow | Verification to execute |
| --- | --- | --- |
| US-01-AC01 | [Identity login](identity.md) | Valid provisioned admin can use all management routes. |
| US-01-AC02 | [Identity login](identity.md) | Unknown user and wrong password have the same safe failure. |
| US-01-AC03 | [Identity guard](identity.md) | Anonymous/non-admin exercise every protected route; no read/write. |
| US-01-AC04 | [Identity logout](identity.md) | Replay old cookie after committed logout; expect 401. |
| US-02-AC01 | [Customers write/read](customers.md) | Create valid client; stable ID appears in list and detail. |
| US-02-AC02 | [Customers validation](customers.md) | Blank name and malformed email; unchanged rows. |
| US-02-AC03 | [Customers update](customers.md) | Edit name/email; preserve ID and linked payment. |
| US-02-AC04 | [Customers association](customers.md) | Same email creates distinct IDs; billing uses selected ID. |
| US-03-AC01 | [Catalog write](service-catalog.md) | 125.00 USD persists as 12500 cents and displays correctly. |
| US-03-AC02 | [Catalog validation](service-catalog.md) | Empty name/nonpositive/excess precision leave record unchanged. |
| US-03-AC03 | [Catalog deactivation/capture](service-catalog.md) | Old history remains; inactive service blocked before Stripe. |
| US-03-AC04 | [Catalog snapshot](service-catalog.md) | Existing request retains price; new request uses edited price. |
| US-04-AC01 | [Billing B1](billing.md) | Persist required snapshot/IDs/time and return test Checkout URL. |
| US-04-AC02 | [Billing B1](billing.md) | Tampered client price rejected; provider receives trusted amount. |
| US-04-AC03 | [Billing B1](billing.md) | Unknown customer/inactive service rejected without provider call. |
| US-04-AC04 | [Billing B1/B2](billing.md) | Concurrent same key, response timeout and crash reuse one session; different input conflicts. |
| US-04-AC05 | [Billing B2](billing.md) | Provider outage returns retryable result with same identity and no Paid. |
| US-05-AC01 | [Billing B3](billing.md) | Hosted link works without login and displays frozen service/USD price. |
| US-05-AC02 | [Billing B3/B4](billing.md) | Successful test payment returns; only verified webhook makes Paid. |
| US-05-AC03 | [Billing B3](billing.md) | Decline leaves Pending; later success yields one Paid purchase. |
| US-05-AC04 | [Billing B3](billing.md) | Cancel, browser close and altered return query cannot mark Paid. |
| US-05-AC05 | [Billing public boundary](billing.md) | Checkout link cannot authorize admin or other payment reads. |
| US-06-AC01 | [Billing B4](billing.md) | Valid signature, correlation and paid evidence record references and PaidAt. |
| US-06-AC02 | [Billing B4](billing.md) | Missing/bad signature and changed raw bytes return 400 with no mutation. |
| US-06-AC03 | [Billing B4](billing.md) | Unknown request/wrong environment/amount/currency quarantine; safe diagnostic. |
| US-06-AC04 | [Billing B4](billing.md) | Unsupported signed event acknowledged; payment unchanged. |
| US-07-AC01 | [Billing B4/B6](billing.md) | Concurrent identical deliveries cause one committed transition. |
| US-07-AC02 | [Billing B4/B5](billing.md) | Distinct events for same session/intent retain one Paid result. |
| US-07-AC03 | [Billing B4/B5](billing.md) | Older expiry/failure cannot downgrade Paid. |
| US-07-AC04 | [Billing B4/B6](billing.md) | Inject DB failure before commit; no 2xx; retry completes atomically. |
| US-07-AC05 | [Billing B6](billing.md) | Five-minute outage; replay twice; one Paid within 60 seconds of receipt. |
| US-08-AC01 | [Billing B7](billing.md) | Mixed states and empty list display required fields correctly. |
| US-08-AC02 | [Billing B7](billing.md) | Detail contains request/provider references, status and confirmed time. |
| US-08-AC03 | [Billing B3/B7](billing.md) | Return without event remains Pending; no browser authority. |
| US-08-AC04 | [Billing B7](billing.md) | Customer edit preserves ID; service edit preserves original snapshot. |
| US-08-AC05 | [Billing B7 / NFR-01](billing.md) | Measure specified staging load; no reporting/export dependency. |

## Functional requirements
| Requirement | Owner / coverage |
| --- | --- |
| FR-01 | Identity; US-01 plus guard on all administrative endpoints |
| FR-02 | Customers; US-02 |
| FR-03 | Service Catalog; US-03 |
| FR-04 | Billing B1–B3; US-04/05 |
| FR-05 | Billing B4–B7; US-06/07/08 |

## Nonfunctional requirements and operations
| Target | Design provision | Required evidence |
| --- | --- | --- |
| NFR-01: list/detail p95 <= 2 seconds | Bounded pages, indexed projection, one batch customer lookup, no provider calls in read path | Record staging CPU/memory/database tier, region, build and dataset; 1,000 customers/10,000 payments, 10 concurrent users, one-minute warmup then five-minute equal list/detail workload across customer/service/payment views; report each route p95 and errors. Measure Stripe creation latency separately. |
| NFR-02: access, webhook and secret protection | Identity guard/CSRF; signed raw-body verification; safe DTOs; secret configuration | Anonymous/non-admin route inventory tests, forged CSRF/signatures, disabled/revoked sessions; scan source/config/database/logs for credentials and card data; inspect HTTPS/cookie settings. Identity password hashes are necessary protected authentication records, never plaintext passwords. |
| NFR-03: 99.5% availability and recovery | One deployable app; minimal readiness check including SQL, separate Stripe dependency metric; durable events and request intents | Probe once/minute; successful eligible probes divided by all eligible probes, excluding declared maintenance. Report monthly results only after deployment. Restore after five-minute outage and resend success twice; verify one row, one transition within 60 seconds of receipt. |

GET /health/ready returns only 200 healthy or 503 unavailable after checking app/SQL readiness. It exposes no configuration. Stripe outages are a separate dependency metric rather than hidden as successful payment creation. Monthly availability is a target, not a promise established by diagrams.

Log only correlation/request/event IDs, safe reason codes, elapsed times and outcome counts. Redact credentials, cookies, personal contact details, correlation tokens, Checkout URLs and raw Stripe bodies. Alert on repeated provider errors, reconciliation backlog and quarantined events. Keep test secrets in local secret configuration and eventual hosted secrets in a managed secret store; commit neither. SQL connection access follows the same rule.

Run schema migrations once through a controlled release step, then start the host. Fail readiness if schema/configuration is incompatible. Keep prior build and database backup/restore procedure available; do not automatically run destructive downgrade migrations. This is deployment design, not authorization to launch production.

## Boundary and exceptional-path checks
| Scenario | Expected invariant |
| --- | --- |
| Module references / repository access | Business modules cannot reference shared EF context or other module internals; Billing consumes only published Customers/Catalog contracts. |
| Price/deactivation racing request capture | Transaction boundary determines one coherent snapshot; no inactive-after-commit capture or mixed name/price. |
| Concurrent creates with one RequestIdentity | One local purchase and one provider session; payload mismatch is 409. |
| Webhook before create response save | Bind verified session to committed intent; later response cannot downgrade status. |
| Duplicate session/intent associated with another request | Unique index and validation reject mutation; safe diagnostic. |
| Crash before Stripe call / after Stripe call / after DB commit | Resume stored intent; same provider key within safe window; old ambiguous requests reconcile without blind recreation. |
| Cached provider error / idempotency window elapsed | Hold and reconcile; never silently rotate the key. |
| DB failure saving unsupported/quarantined receipt | Return 503 rather than claiming durable handling. |
| Commit succeeds but response is lost | Redelivery sees committed receipt and acknowledges without mutation. |
| Provider retrieval unavailable during webhook | 503; no processed receipt; later delivery can complete. |
| Concurrent updates to customer/service | Expected version mismatch gives 409, no silent overwrite. |
| App restart | Persisted sessions/key material and request recovery state survive; expired leases become reclaimable. |

## Implementation test layers
- Domain checks: amount precision, snapshot immutability, allowed state transitions and identity-conflict rules.
- SQL Server integration checks: transactional event/payment commit, unique keys, concurrent requests/events, restricted foreign keys and row-version conflicts.
- HTTP integration checks: route authorization, CSRF, safe errors, raw-body webhook signature validation and DTO data exposure.
- Stripe test-mode end-to-end checks: create link, successful/declined card, cancellation, real signed delivery/resend and provider-session correlation.
- Load and outage checks: the three NFR scenarios above with captured evidence.

## Scope and review boundary
No model or workflow is required for recurring memberships, subscriptions, invoices, refunds or advanced reporting because those are explicit exclusions. No customer login, email delivery, scheduling or health-data module is introduced. The public return page is not a customer portal.

This package covers the existing eight stories without claiming implemented behavior. Mentor approval and acceptance-test execution remain pending. No public repository settings, branch protection, PR merges, infrastructure or runtime tests are verified by this document.
