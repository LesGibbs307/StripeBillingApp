# B5 — Payment state machine

[Diagram index](README.md) | [Mermaid source](13-payment-states.mmd) | [SVG](rendered/13-payment-states.svg) | [PNG](rendered/13-payment-states.png) | [Full workflow context](../billing.md)

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
