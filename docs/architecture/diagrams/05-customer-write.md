# Write workflow

[Diagram index](README.md) | [Mermaid source](05-customer-write.mmd) | [SVG](rendered/05-customer-write.svg) | [PNG](rendered/05-customer-write.png) | [Full workflow context](../customers.md)

```mermaid
flowchart TD
    A["Authorized Admin + CSRF"] --> V{"Name and email valid?"}
    V -->|No| E["400 field errors; no write"]
    V -->|Yes| T{"Create or update?"}
    T -->|Create| N["Assign server CustomerId"]
    T -->|Update| F{"Customer exists?"}
    F -->|No| NF["404"]
    F -->|Yes| C{"Expected row version matches?"}
    C -->|No| CF["409; reload before editing"]
    C -->|Yes| U["Change name/email only; preserve ID"]
    N --> D["Commit Customer transaction with atomic row-version check"]
    U --> D
    D --> S{"Commit outcome?"}
    S -->|Version conflict| CF
    S -->|Other failure| ER["503; no success response"]
    S -->|Success| O["201 create or 200 update with ID and row version"]
```
