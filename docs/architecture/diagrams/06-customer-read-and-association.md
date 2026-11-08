# Read and association workflow

[Diagram index](README.md) | [Mermaid source](06-customer-read-and-association.mmd) | [SVG](rendered/06-customer-read-and-association.svg) | [PNG](rendered/06-customer-read-and-association.png) | [Full workflow context](../customers.md)

```mermaid
flowchart TD
    R["Authorized list/detail request"] --> P{"Pagination and ID valid?"}
    P -->|No| Invalid["400; no query"]
    P -->|Yes| Q["Read only customer-owned records"]
    Q -->|Database failure| Down["503; no success data"]
    Q --> O{"Found?"}
    O -->|List empty| E["200 explicit empty list"]
    O -->|Detail missing| N["404"]
    O -->|Yes| D["200 ID, name, email and row version"]
    B["Billing asks RequireCustomer by ID"] --> K["Read existing stable ID in shared scoped transaction"]
    K -->|Database failure| Down
    K --> C{"Exists?"}
    C -->|No| X["Reject payment creation before Stripe"]
    C -->|Yes| Y["Return customer DTO; never infer identity from email"]
```
