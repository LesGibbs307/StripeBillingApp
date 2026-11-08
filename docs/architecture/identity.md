# Identity module
[Architecture index](README.md) · **Owner:** US-01, NFR-02

## Contract and access policy
Provision staff through a controlled administrative command during setup, outside public HTTP routes. Assign the single Admin role using ASP.NET Core Identity. Never commit a seed password. Disable public registration and password-reset email flows in this MVP; a controlled operator procedure can reset a demo account.

Endpoints:
| Endpoint | Access / behavior |
| --- | --- |
| GET /api/auth/csrf | Anonymous; issues antiforgery material, no business data |
| POST /api/auth/login | Anonymous, CSRF required, rate limited; generic failure |
| POST /api/auth/logout | Current session + CSRF; revoke server-side before success |
| GET /api/auth/me | Authenticated Admin; minimal actor information |

Use a secure, HttpOnly, SameSite=Lax authentication cookie, HTTPS outside local development, and a server-side AdminSession ticket store. No browser local-storage bearer token. Store a hash of the random session ticket; retain a 30-minute absolute expiry without sliding renewal. Validate ticket expiry, revocation, enabled user, current Admin membership and Identity security stamp on every protected request. Persist data-protection keys securely across host restarts. These are proposed policies, not existing configuration.

All /api/customers, /api/services and /api/billing routes require Admin authorization. Anonymous/expired sessions receive 401; authenticated non-admins receive 403. API failures do not redirect to a login HTML page. Unsafe cookie-authenticated requests require CSRF protection; the Stripe webhook instead uses signature authentication. Explicit exceptions are login/CSRF bootstrap, static return/cancel pages and a minimal health endpoint.

## Provisioning and login workflow
```mermaid
flowchart TD
    P["Operator supplies demo credentials securely"] --> U["Identity creates hashed password and Admin role membership"]
    U --> Ready["Provisioned account ready; setup completes"]
    L["Login submitted with CSRF token"] --> Store{"Identity store available?"}
    Store -->|No| Down["503; no session"]
    Store -->|Yes| V
    V{"CSRF valid and rate limit available?"}
    V -->|No| R["400 or 429; no session"]
    V -->|Yes| C{"Credentials valid, enabled Admin, not locked?"}
    C -->|No| G["Generic 401; record sanitized failed attempt"]
    C -->|Yes| S["Persist random session ticket hash and expiry"]
    S --> DB{"Commit succeeded?"}
    DB -->|No| E["503; do not issue authenticated cookie"]
    DB -->|Yes| OK["Issue cookie; allow admin navigation"]
```

Reject unknown users and incorrect passwords with the same message. Use Identity's password hashing and lockout support; proposed policy: five failed attempts trigger a 15-minute lockout, with an additional IP rate limit. Do not log passwords, cookies, antiforgery tokens or credential-bearing request bodies. Validation responses must not reveal whether a username exists.

## Guard and logout workflow
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

A request already executing before revocation may finish; any request authorized after committed logout must reject the old ticket. Database unavailability fails closed with 503 rather than treating an unvalidated session as authenticated. Identity records store password hashes, never plaintext passwords; the requirements' credentials prohibition applies to plaintext credentials, API keys and secret tokens, not necessary authentication hashes.

Verification: exercise valid/invalid login, lockout, all module endpoints as anonymous/non-admin, CSRF rejection, expiry, disabled users and replay of a captured pre-logout cookie. See the [coverage matrix](coverage.md).
