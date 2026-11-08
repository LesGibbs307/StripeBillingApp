# B7 — Admin payment queries

[Diagram index](README.md) | [Mermaid source](15-payment-queries.mmd) | [SVG](rendered/15-payment-queries.svg) | [PNG](rendered/15-payment-queries.png) | [Full workflow context](../billing.md)

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
