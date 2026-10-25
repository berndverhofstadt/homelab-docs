---
authors:
    - Bernd Verhofstadt
weight: 50
---
# Authentication and access control

Routing and authentication are separate responsibilities. Traefik routes
requests to an application; each service's access policy must be chosen and
verified independently. Some services use their own accounts or native
OpenID Connect, while selected routes can use forward authentication.

The Compose configuration uses Pocket ID as an OpenID Connect provider and
Tinyauth as an optional forward-auth bridge for Traefik. The middleware is
opt-in per router, not applied globally. A service that does not have an
authentication gate must be limited by its intended network exposure; a
hostname or TLS certificate is not authentication.

Authentik is deferred in the current deployment design. The
[identity-provider guide](IdP.md) describes the current Pocket ID/Tinyauth
pattern and general OIDC integration.

Before exposing a service, check that its own login, proxy middleware,
identity integration and public/LAN exposure match its intended use. In
particular, do not publish LAN-only services through a public tunnel.
