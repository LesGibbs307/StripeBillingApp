# B6 — Outage and acknowledgement recovery

[Diagram index](README.md) | [Mermaid source](14-outage-recovery.mmd) | [SVG](rendered/14-outage-recovery.svg) | [PNG](rendered/14-outage-recovery.png) | [Full workflow context](../billing.md)

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
