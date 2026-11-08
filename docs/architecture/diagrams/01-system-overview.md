# System diagram

[Diagram index](README.md) | [Mermaid source](01-system-overview.mmd) | [SVG](rendered/01-system-overview.svg) | [PNG](rendered/01-system-overview.png) | [Full workflow context](../README.md)

```mermaid
flowchart TB
    Admin["Fitness administrator"] --> UI["React UI running in browser"]
    Client["Fitness client"] --> Checkout["Stripe hosted Checkout"]
    subgraph App["RepRise — one ASP.NET Core process / deployment"]
        Assets["Serves compiled React assets"]
        Guard["Authentication, authorization and CSRF"]
        Identity["Identity module"]
        Customers["Customers module"]
        Catalog["Service Catalog module"]
        Billing["Billing module"]
        CC["Customers public contract"]
        SC["Catalog public contract"]
        Webhook["Billing webhook endpoint"]
        Adapter["Billing Stripe API adapter"]
        Return["Public static return / cancellation pages"]
        Guard --> Identity
        Guard --> Customers
        Guard --> Catalog
        Guard --> Billing
        Billing --> CC --> Customers
        Billing --> SC --> Catalog
        Webhook --> Billing
        Billing --> Adapter
    end
    Assets -.-> UI
    UI --> Guard
    Adapter --> API["External Stripe API"]
    API -.-> Checkout
    Checkout --> Return
    Events["Stripe event delivery"] -->|Signed HTTP POST| Webhook
    Identity --> SQL[("SQL Server — one database")]
    Customers --> SQL
    Catalog --> SQL
    Billing --> SQL
```
