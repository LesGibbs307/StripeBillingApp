# Guard and logout workflow

[Diagram index](README.md) | [Mermaid source](04-authorization-and-logout.mmd) | [SVG](rendered/04-authorization-and-logout.svg) | [PNG](rendered/04-authorization-and-logout.png) | [Full workflow context](../identity.md)

```mermaid
flowchart TD
    Q["Protected request"] --> DB{"Session store available?"}
    DB -->|No| Error["503; fail closed"]
    DB -->|Yes| S{"Ticket current and user enabled?"}
    S -->|No| A["401; no business handler runs"]
    S -->|Yes| R{"Current Admin role?"}
    R -->|No| F["403; no business data"]
    R -->|Yes| C{"Unsafe method?"}
    C -->|Yes| T{"CSRF valid?"}
    T -->|No| X["400; no mutation"]
    T -->|Yes| L{"Logout?"}
    C -->|No| H["Run authorized read handler"]
    L -->|No| W["Run authorized mutation handler"]
    L -->|Yes| D["Persist session revocation"]
    D --> K{"Commit succeeded?"}
    K -->|No| E["503; do not claim successful logout"]
    K -->|Yes| O["Clear cookie; 204; old ticket rejected"]
    H --> B["Return module result"]
    W --> B
```
