---
authors:
    - Bernd Verhofstadt
weight: 51
---
# Identity Provider: authentik

While Authelia and Traefik protect individual web applications at the reverse-proxy layer, I also run a separate **Identity Provider (IdP)** based on **authentik**. This IdP provides modern protocols (OpenID Connect and SAML) and centralizes user accounts, groups and single sign-on across multiple services.

## Why an IdP in a homelab?

Having a dedicated IdP brings several benefits, even in a home environment:

- **Single Sign-On (SSO)** – Sign in once at the IdP and access multiple apps without re‑entering credentials.
- **Central user management** – Users, groups and MFA policies are managed in one place.
- **Standard protocols** – Many modern services support OpenID Connect or SAML out of the box.
- **Clean separation of concerns** – The IdP handles identity; Traefik/Authelia handle HTTP routing and access control.

In my setup, authentik is the source of truth for identities. Some applications integrate with it directly via OpenID Connect, while others are protected by Authelia in front of Traefik.

## High‑level architecture

At a conceptual level, the authentication stack looks like this:

- **authentik (IdP)**  
    - Runs as a Docker container on Unraid.  
    - Uses PostgreSQL and Redis as backend services.  
    - Exposes a web interface at a hostname like `id.example.com`.
- **Traefik (reverse proxy)**  
    - Terminates HTTPS and routes requests to backend services.  
    - For some apps, Traefik forwards authentication to Authelia; for others, it simply passes through to applications that speak OpenID Connect/SAML and rely on authentik directly.
- **Authelia (access portal)**  
    - Acts as a front‑door for selected sensitive services (see Authentication overview page).  
    - Uses its own user store or can delegate to upstream identity providers.

Applications then fall into two categories:

1. **Apps that support OpenID Connect/SAML** – These are configured as clients of authentik. Users are redirected to `id.example.com` for login, and upon success they are sent back with tokens.
2. **Apps without IdP support** – These are placed behind Traefik + Authelia. Authelia handles login and authorization before the request ever reaches the app.

## Running authentik on Unraid

authentik runs as a container with:

- A **configuration volume** for templates and media (e.g. custom emails, logos)
- Access to the **Docker socket** so it can discover applications if desired
- Environment variables that point to:
    - PostgreSQL (`AUTHENTIK_POSTGRESQL__HOST`, user, database)
    - Redis (`AUTHENTIK_REDIS__HOST`)
    - Email settings (SMTP host, port, username, sender)

In front of the container, Traefik is configured with labels similar to:

```text
traefik.enable=true
traefik.http.routers.authentik.rule=Host(`id.example.com`)
traefik.http.routers.authentik.entryPoints=https
traefik.http.services.authentik.loadbalancer.server.port=9000
```

> **Sanitised example**  
> In the real configuration, hostnames, domains and credentials are different. For public documentation, everything is reduced to placeholders like `id.example.com` and `<SMTP_PASSWORD>`.

## Integrating applications via OpenID Connect

To onboard an application to authentik using OpenID Connect, the general steps are:

1. **Create a provider** in authentik (OpenID Connect or OAuth2).  
     - Define redirect URLs like `https://app.example.com/auth/callback`.  
     - Configure scopes (e.g. `openid`, `profile`, `email`).
2. **Create an application** and link it to the provider.  
     - Assign which user groups may access the application.  
     - Optionally configure rules (e.g. require MFA).
3. **Configure the client app**:  
     - Set the issuer URL (e.g. `https://id.example.com/application/o/app/`).  
     - Enter client ID and client secret from authentik.  
     - Match redirect/callback URLs.

After this, the app delegates login to authentik. Users see a consistent login page and can reuse the same MFA method for multiple services.

## Relationship between authentik and Authelia

In this homelab I use both components, but for different roles:

- **authentik** – Primary Identity Provider for applications that speak OpenID Connect/SAML.
- **Authelia** – Access gateway in front of Traefik for apps that have no native IdP integration or where I want a very lightweight protection.

This combination keeps the authentication architecture flexible: some services get full SSO via authentik, while others are simply shielded behind Authelia without needing any changes inside the application itself.