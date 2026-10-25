---
authors:
    - Bernd Verhofstadt
weight: 51
---
# Identity provider: Pocket ID and OIDC

Pocket ID provides OpenID Connect (OIDC) identity for applications that
support it. Traefik routes traffic, while Tinyauth can bridge OIDC login to
Traefik's forward-auth middleware for selected applications. The middleware
is opt-in per route; the identity provider is not placed in front of itself.

## Components

- **Pocket ID** – OIDC provider with persistent application data.
- **Tinyauth** – optional forward-auth service that redirects unauthenticated
  users through Pocket ID.
- **Traefik** – routes requests and invokes forward-auth only for routers
  explicitly configured to use it.

The Compose deployment keeps Pocket ID and Tinyauth on `proxy`. Do not mount
the raw Docker socket into an identity service. Keep encryption keys, OIDC
client secrets and other credentials in protected environment files or a
secret manager.

## Protect a route with forward auth

Add the authentication middleware only to routes that should require it:

```text
traefik.http.routers.example-app.rule=Host(`app.example.org`)
traefik.http.routers.example-app.entrypoints=https
traefik.http.routers.example-app.middlewares=pocketid-auth@file
```

Create an OIDC client in Pocket ID using the callback URL documented by
Tinyauth, for example `https://auth.example.org/api/oauth/callback/provider`.
Set the issuer, client ID and client secret in the protected configuration.
Use the exact callback and issuer URLs required by the deployed software.

## Native OIDC applications

For an application with native OIDC support:

1. Create an OIDC client in Pocket ID.
2. Register the application's exact redirect URI, such as
   `https://app.example.org/auth/callback`.
3. Configure the issuer URL, client ID, client secret and requested scopes in
   the application.
4. Verify both successful login and denied access for an unauthorized
   account.

## Access policy

Authentication is not universal: application login, OIDC and forward auth
are selected per service. Review the route configuration before assuming a
service is protected. Keep services intended to be LAN-only out of public
tunnel ingress and verify that restriction from an off-LAN client.

Authentik and Authelia examples from earlier iterations are not part of the
current Compose authentication design.
