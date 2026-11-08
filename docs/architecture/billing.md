# Billing module
[Architecture index](README.md) · **Owner:** US-04 through US-08, FR-04/05

Owns a PaymentRequest aggregate and WebhookEvent receipts. One request is one purchase of one service, quantity one. A separate Payment table is unnecessary for the MVP: the request stores the single successful provider payment reference. Do not create Subscription, Plan, Membership, Invoice or Refund models.

## API and public boundary
| Route | Access | Result |
| --- | --- | --- |
| POST /api/billing/payment-requests | Admin + CSRF | Create/resume using RequestIdentity, CustomerId, ServiceId |
| GET /api/billing/payment-requests | Admin | Paginated payment summaries |
| GET /api/billing/payment-requests/{id} | Admin | Persisted detail and available Checkout URL |
| POST /webhooks/stripe | Stripe signature; no cookie/CSRF | Process selected snapshot events |
| GET /checkout/return and /checkout/cancel | Public static pages | Generic message, no business data or state mutation |

Reject client-supplied price/currency fields. The React form generates a random RequestIdentity once per intentional purchase and retains it across retries and reloads. Disable double-submit as a convenience; database constraints supply correctness. Same identity plus different customer/service returns 409. Admins may access all company requests; identity reuse is checked globally rather than generating a new key per staff member.

Common responses: 400 validation, 401/403 authorization, 404 missing record, 409 conflict, 202 still creating/reconciling, 503 temporary dependency failure. Responses carry RequestId when already allocated and a safe error code; never claim Paid on error. No stack traces or provider secrets.

## B1 — Capture and create Checkout
```mermaid
flowchart TD
    A["Authorized create/resume with stable RequestIdentity"] --> Input{"Input valid; no client price/currency?"}
    Input -->|No| Bad["400; no provider call"]
    Input -->|Yes| E{"Request identity already exists?"}
    E -->|Yes| P{"Customer/service fingerprint matches?"}
    P -->|No| X["409 identity conflict"]
    P -->|Yes| Ready{"Known session?"}
    Ready -->|Yes| Same["Return original state and available URL; no replacement"]
    Ready -->|No| Retry["B2 safe retry/reconciliation; no direct new POST"]
    E -->|No| T["Begin short database transaction"]
    T --> C["Validate customer and capture locked active service through contracts"]
    C --> V{"Customer, service and charge bounds valid?"}
    V -->|No| R["Rollback; validation response; no Stripe call"]
    V -->|Yes| Save["Persist Pending intent, snapshot, frozen parameters and key"]
    Save --> Commit{"Commit outcome?"}
    Commit -->|Unique identity conflict| Reload["Rollback and load winning request"]
    Reload --> P
    Commit -->|Other failure| Fail["503; no provider call"]
    Commit -->|Success| Lease["Acquire per-request database lease"]
    Lease --> Win{"Lease acquired?"}
    Win -->|No| Wait["202; preserve identity"]
    Win -->|Yes| First["Durably record FirstAttemptAt before network call"]
    First --> Stored{"Persisted?"}
    Stored -->|No| Fail
    Stored -->|Yes| Stripe["Call Stripe with frozen payload and stored key"]
    Stripe --> Response{"Outcome?"}
    Response -->|Timeout or transient error| Amb["Keep current payment state; resume through B2"]
    Response -->|Definitive invalid parameters| Invalid["Persist Rejected creation status; no Paid inference"]
    Response -->|Session returned| Attach["Lock request; validate response, lease and session binding"]
    Attach --> Bind{"Binding null or same, and response valid?"}
    Bind -->|No| Hold["Preserve existing record; flag reconciliation"]
    Bind -->|Yes| Write["Attach session and URL; never downgrade payment status"]
    Write --> Done{"Commit succeeds?"}
    Done -->|No| Recover["503; B2 or webhook can recover"]
    Done -->|Yes| URL["201 first creation or 200 resume with original state"]
```

Create the request and capture the service in one transaction. A unique RequestIdentity constraint handles racing submissions: the loser rolls back, reloads the winner and compares the fingerprint. Lock ordering is customer, service, then payment request for new creation; webhook transactions touch only Billing.

The persisted Stripe payload uses mode=payment, card payment methods, quantity=1, inline price_data with the frozen USD amount and service name, fixed return/cancel URLs, client_reference_id=RequestId, and metadata RequestId plus a random correlation token. Disable promotion codes, automatic tax and adjustable quantity. No customer email is required in this payload; Stripe handles payment entry. Any provider parameters used must be frozen before the first call.

Record FirstAttemptAt before calling Stripe. A short database lease serializes normal attempts; a 60-second lease with owner token is renewed while active and expires after a crashed host. Lease fencing prevents stale workers overwriting newer results. Stripe idempotency still protects overlapping calls after lease expiry. Attaching a response must never reset an already Paid/Expired status. If a webhook bound the same session first, attaching is a no-op; a different session is quarantined for operator review.

## B2 — Ambiguous creation and recovery
Stripe API v1 may retain the first response including an error, and keys can be removed after 24 hours. Therefore indefinite resubmission is unsafe. See [Stripe idempotency semantics](https://docs.stripe.com/api/idempotent_requests).

```mermaid
flowchart TD
    A["Resume request missing Checkout result"] --> Lease{"Acquire persisted lease?"}
    Lease -->|No| Wait["202; preserve identity"]
    Lease -->|Yes| K{"Known session after reload?"}
    K -->|Yes| R["Return original session/state"]
    K -->|No| Rej{"Creation definitively rejected?"}
    Rej -->|Yes| Stop["Show creation error; operator resolves; no automatic changed payload"]
    Rej -->|No| First{"FirstAttemptAt exists?"}
    First -->|No| Mark["Persist FirstAttemptAt before first POST"]
    Mark --> OK{"Commit succeeds?"}
    OK -->|No| Error["503; no provider call"]
    OK -->|Yes| Retry["Call with identical frozen payload and original key"]
    First -->|Yes| W{"Under 23 hours and no cached server-error ambiguity?"}
    W -->|Yes| Retry
    W -->|No| Rec["Persist NeedsReconciliation"]
    Retry --> O{"Outcome?"}
    O -->|Session| Bind["Validate session, lock request and check binding"]
    O -->|Definitive rejection| Stop
    O -->|Transient or ambiguous| Rec
    Rec --> Find["Read provider sessions; fully paginate attempt window; match ID and token"]
    Find --> M{"Exactly one verified match?"}
    M -->|Yes| Bind
    M -->|No or provider unavailable| Hold["Keep unresolved; safe diagnostic; no new Checkout POST"]
    Bind --> Valid{"Valid and binding null or same?"}
    Valid -->|No| Hold
    Valid -->|Yes| Save["Attach session and Ready creation status; preserve payment state"]
    Save --> Commit{"Commit succeeds?"}
    Commit -->|No| Error
    Commit -->|Yes| R
```

Use a Billing in-process recovery task every 30 seconds, with persisted leases and bounded exponential backoff (1, 2, 4 seconds for immediate transient retries; never endless retries on a web request). Reconciliation uses read-only Stripe API calls through the same configured test account and validates the entire frozen purchase. Never log the correlation token. No raw Checkout payloads are stored in logs.

Zero matches does not prove a failed creation when provider history/visibility is incomplete; multiple matches indicate an invariant breach. Keep these requests blocked and surface an admin-safe “creation needs reconciliation” message. The operator checks Stripe test-mode records and request IDs; resolve only by binding a proven matching session or recording a confirmed failure with evidence. Do not mint a fresh provider key automatically. A new intentional purchase uses a new RequestIdentity; the UI must not suggest a new purchase as a timeout retry.

A definitively rejected provider payload remains on the original request with a safe creation error. Correct configuration before resuming; never mutate parameters for an identity already sent. If purchase terms must change, the original must be conclusively nonpayable before a distinct purchase is created. This operational exception does not add a refund or payment-management feature.

## B3 — Hosted payment and browser return
```mermaid
flowchart TD
    L["Client opens manually shared Checkout link"] --> S["Stripe displays frozen service / USD amount"]
    S --> A{"Payment outcome"}
    A -->|Successful card payment| W["Stripe emits signed event independently of browser"]
    W --> H["B4 verifies and records payment"]
    A -->|Declined card| D["Stripe shows decline; no app state change"]
    D --> S
    A -->|Cancel or close browser| P["No app state change; no success or expiry inferred"]
    A -->|Session unavailable or expired| E["Stripe rejects payment; verified expiry may set Expired"]
    A -->|Return to app| R["Static confirmation-processing or cancellation message"]
    R --> N["Ignore query parameters for payment state; expose no business data"]
    W -.->|Client may return| R
```

Clients need no app login. Checkout links grant no admin permissions. Return/cancel pages expose no customer details, request history or privileged API tokens. Return URLs do not carry business data. The payment view remains Pending unless a verified event has already changed it; a browser return must never overwrite a legitimately Paid result. No appointment, membership entitlement or email is produced.

## B4 — Webhook verification and atomic processing
Subscribe to snapshot events checkout.session.completed and checkout.session.expired in the configured test account. Cards only; asynchronous payment methods are disabled, so adding them later requires new event handling and acceptance tests.

Verify the Stripe-Signature using the SDK against the original raw request bytes, with its normal timestamp tolerance and a dedicated endpoint secret. Keep the host clock synchronized. Stripe documents raw-body verification, possible duplicates and unordered delivery in its [webhook guide](https://docs.stripe.com/webhooks).

```mermaid
flowchart TD
    A["Raw bytes and Stripe signature"] --> Sig{"Signature valid?"}
    Sig -->|No| Reject["400; no Billing mutation"]
    Sig -->|Yes| Read["Read durable event receipt"]
    Read --> DB{"Database available?"}
    DB -->|No| Retry["503; no successful acknowledgement"]
    DB -->|Yes| Dup{"Already handled?"}
    Dup -->|Yes| Ack["200; no second update"]
    Dup -->|No| Type{"Supported event?"}
    Type -->|No| Ignore["Prepare Ignored receipt"]
    Type -->|Yes| Lookup{"Known request and expected test environment/account?"}
    Lookup -->|No| Quar["Prepare Quarantined receipt and safe diagnostic"]
    Lookup -->|Yes| Retrieve["Retrieve current session and expanded PaymentIntent"]
    Retrieve --> Available{"Provider result?"}
    Available -->|Transient failure| Retry
    Available -->|Object invalid or absent| Quar
    Available -->|Available| Match{"Token, session, mode, amount and currency match?"}
    Match -->|No| Quar
    Match -->|Yes| TX["Begin transaction; lock request and recheck receipt/binding"]
    TX --> Recheck{"Concurrent result?"}
    Recheck -->|Receipt now handled| Existing["End transaction without mutation"]
    Existing --> Ack
    Recheck -->|Binding conflict| Rollback["Rollback; prepare quarantine diagnostic"]
    Rollback --> Quar
    Recheck -->|Safe to proceed| Bind["Bind verified session if missing; enforce unique provider IDs"]
    Bind --> Success{"Session paid and successful intent amount/currency match?"}
    Success -->|Yes| Paid["Set Paid once; preserve original PaidAt"]
    Success -->|No| Exp{"Current session expired and request not Paid?"}
    Exp -->|Yes| Expired["Set Expired"]
    Exp -->|No| Noop["No payment-state change"]
    Paid --> Receipt["Insert event receipt in payment transaction"]
    Expired --> Receipt
    Noop --> Receipt
    Receipt --> Commit{"Commit outcome?"}
    Commit -->|Success| Ack
    Commit -->|Duplicate key| Resolve["Rollback; read committed event receipt"]
    Commit -->|Other failure| Retry
    Resolve --> Found{"Handled receipt exists?"}
    Found -->|Yes| Ack
    Found -->|No| Retry
    Ignore --> Durable["Commit diagnostic receipt without payment mutation"]
    Quar --> Durable
    Durable --> Saved{"Commit outcome?"}
    Saved -->|Success| Ack
    Saved -->|Duplicate key| Resolve
    Saved -->|Other failure| Retry
```

Correlation rules:
- Require the local RequestId, persisted correlation token, current session metadata and frozen amount/currency to agree. Validate event/session livemode=false and use only the configured account's credentials. Connect/multi-account events are not supported.
- A stored CheckoutSessionId must equal the incoming/retrieved ID. If it is null because the webhook won the race, bind only after all correlation checks succeed against the already committed request.
- For Paid, require mode=payment, payment_status=paid, successful PaymentIntent status, matching PaymentIntent currency/received amount, and the session's reference to that intent. Enforce unique session and payment-intent IDs in SQL.
- PaidAt is the first local UTC verified-confirmation time, not a claimed bank settlement timestamp. Preserve it on all later deliveries.
- Current Stripe state takes precedence over an older event snapshot. Pending or declined evidence never downgrades a paid request. Unknown or mismatched events cannot bind a session.
- Unsupported signed events receive an Ignored receipt. A known supported event that cannot be trusted receives a Quarantined receipt with a reason code and safe IDs only. These are deliberately acknowledged after durable recording to avoid poison-event retry loops; operator investigation is required.
- Transient provider/database failures receive 503 without a processed receipt. An operator reprocesses a quarantined event only after resolving the cause, using the same validation pipeline and an explicitly reopened receipt; payment guards still apply.

Concurrency: lock the PaymentRequest row before state changes; insert a uniquely indexed account/environment/EventId receipt in the same transaction. On a duplicate-key race, roll back and read the committed receipt: acknowledge only if it exists. Distinct event IDs for the same session serialize on the request and produce one Paid transition. Bound deadlock/row-version retries; return 503 when exhausted. Never mark a receipt Processed before the payment update is durable.

## B5 — Payment state machine
CreationStatus (Creating, Ready, NeedsReconciliation, Rejected) describes link creation and is separate from payment Status.
```mermaid
stateDiagram-v2
    [*] --> Pending: Request persisted
    Pending --> Pending: Decline / browser cancel / return / transient failure
    Pending --> Paid: Verified current payment success
    Pending --> Expired: Verified current session expiry
    Expired --> Paid: Verified authoritative success wins
    Expired --> Expired: Duplicate expiry / browser return / no success evidence
    Paid --> Paid: Duplicate / older expiry / other later event
```
No transition out of Paid in this MVP. Refunds are future scope. A delayed expiry event must retrieve current state before acting. Expired requests are retained; retrying their original identity does not create another session.

## B6 — Outage and acknowledgement recovery
```mermaid
sequenceDiagram
    participant S as Stripe / test resend
    participant H as Billing host
    participant D as SQL
    S->>H: Delivery during five-minute outage
    Note over H: Unavailable, no successful acknowledgement
    Note over H: Restore host, configuration and database
    S->>H: Resend valid success event with fresh signature
    Note over H: Verify fresh signature, then check durable receipt
    H->>S: Retrieve authoritative session
    S-->>H: Session and payment evidence
    Note over H: Validate environment, correlation and payment evidence
    H->>D: Atomic request update + receipt
    alt Database commit fails
        D-->>H: Failure
        H-->>S: 503, retry permitted
    else Database commit succeeds
        D-->>H: Durable Paid + receipt
        H-->>S: 200
        S->>H: Duplicate event, including after lost response
        Note over H: Verify duplicate delivery signature first
        H->>D: Read committed receipt
        H-->>S: 200, no second business update
    end
```
Recovery target: within 60 seconds of receipt after restoration, with dependencies available, both replay deliveries yield one payment row and one Paid transition. Use Stripe resend or a fresh valid signed fixture; a five-minute-old raw signature is not an acceptable replay test. This design uses synchronous durable handling, not a queue. Provider calls have timeouts; failure to finish returns non-2xx for later delivery.

## B7 — Admin payment queries
```mermaid
flowchart TD
    A["Authorized list/detail"] --> V{"ID or pagination valid?"}
    V -->|No| Invalid["400; no query"]
    V -->|Yes| B["Read Billing projection from SQL; no Stripe call"]
    B -->|Database failure| Down["503; no successful result"]
    B --> F{"Detail exists / list has rows?"}
    F -->|Missing detail| N["404"]
    F -->|Empty list| E["200 explicit empty list"]
    F -->|Rows| C["Batch customer display summaries by stable IDs"]
    C -->|Dependency failure| Down
    C -->|Success| O["200 current customer, frozen service/amount, status and timestamps"]
    O --> UI["Show verified state; creation issues separately"]
```
Default page size 25, maximum 100; order CreatedAt descending then Id descending. Use bounded projections and a matching index; no per-row Stripe calls or customer queries. Detail includes RequestId, CustomerId, ServiceId, snapshot, currency, status, available session/intent IDs, CreatedAt, PaidAt and creation status; Checkout URL only for authorized staff. A customer edit can change display name, never customer association. Service/price always come from the snapshot. Do not include raw events, credentials, correlation tokens or provider payloads in DTOs.
