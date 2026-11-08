# Create/edit/deactivate workflow

[Diagram index](README.md) | [Mermaid source](07-service-management.mmd) | [SVG](rendered/07-service-management.svg) | [PNG](rendered/07-service-management.png) | [Full workflow context](../service-catalog.md)

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
