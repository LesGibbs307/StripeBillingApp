# Service Catalog module
[Architecture index](README.md) · **Owner:** US-03

Owns Service. A service is a fixed-price personal-training session or assessment; it does not schedule delivery or track membership.

## Endpoints and contract
GET /api/services; GET /api/services/{id}; POST /api/services; PUT /api/services/{id}; POST /api/services/{id}/deactivate. All require Admin, with CSRF for mutations. Updates/deactivation require expected row version.

Input: trimmed nonblank name up to 200 characters; positive decimal USD price with at most two decimal places. Parse using decimal, reject additional precision rather than rounding, then store integer cents. Currency is fixed to USD. The UI may format 12500 cents as $125.00; it cannot determine the charge amount. Persist valid positive prices even if outside Stripe's currently supported charge range; Billing rejects an unchargeable amount before calling Stripe using provider limits verified for the selected API/account at implementation.

ServiceCatalog.Contracts exposes CaptureActiveService(id, transaction), returning ServiceId, name, AmountMinor, currency and version. It holds a database read/update lock until the shared short transaction commits. An edit or deactivation either commits before capture (new values/rejection) or after capture (existing request keeps its snapshot). This defines what “active at creation” means without a race.

## Create/edit/deactivate workflow
```mermaid
flowchart TD
    A["Authorized Admin + CSRF"] --> Op{"Operation?"}
    Op -->|Create| V["Validate name and positive USD price precision"]
    Op -->|Edit| F["Load existing service and check row version"]
    Op -->|Deactivate| F
    F --> M{"Exists and version current?"}
    M -->|Missing| N["404"]
    M -->|Conflict| C["409; reload"]
    M -->|Yes| D{"Deactivate?"}
    D -->|Yes| I["Set IsActive false; preserve record"]
    D -->|No| V
    V --> OK{"Valid?"}
    OK -->|No| E["400; no change"]
    OK -->|Yes| S["Create active service or update existing fields"]
    S --> DB["Commit service transaction with atomic row-version check"]
    I --> DB
    DB --> R{"Commit outcome?"}
    R -->|Version conflict| C
    R -->|Other failure| X["503; no success response"]
    R -->|Success| O["Return saved record; historical requests untouched"]
```

No delete endpoint. Editing a deactivated service does not implicitly reactivate it. Reactivation is not needed for this MVP. Reads use page size 25, maximum 100, deterministic ID ordering; empty list is explicit and missing detail is 404.

## Trusted snapshot workflow
```mermaid
flowchart TD
    B["Billing begins request transaction"] --> L["Catalog locks and reads selected service"]
    L --> A{"Exists and active?"}
    A -->|No| R["Reject; rollback; no Stripe call"]
    A -->|Yes| D["Return trusted name, price, currency and version"]
    D --> P["Billing persists immutable request snapshot"]
    P --> C{"Commit succeeds?"}
    C -->|No| E["Rollback and release lock; no Stripe call"]
    C -->|Yes| S["Release lock; later edits affect only future requests"]
```

Verification: $125.00 round trip; blank/zero/negative/excess-precision rejection; deactivation blocks a new request but retains old records; concurrent capture/deactivation follows the transaction boundary; a later price edit cannot change an existing Checkout amount.
