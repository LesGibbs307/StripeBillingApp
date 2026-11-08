# B3 — Hosted payment and browser return

[Diagram index](README.md) | [Mermaid source](11-hosted-payment.mmd) | [SVG](rendered/11-hosted-payment.svg) | [PNG](rendered/11-hosted-payment.png) | [Full workflow context](../billing.md)

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
