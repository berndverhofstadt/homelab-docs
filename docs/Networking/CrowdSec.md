---
authors:
    - Bernd Verhofstadt
weight: 44
---
# Protecting Traefik with CrowdSec

## At a glance

- Explains how the CrowdSec agent and Traefik bouncer work together to block malicious IPs.
- Shows how to wire both containers on Unraid and connect Traefik via ForwardAuth.
- Highlights how logs and placeholders are used so no real IPs, hosts or keys appear in the docs.

To protect public‑facing services from brute‑force attempts, scanners and other malicious traffic, I use **CrowdSec** in combination with a **Traefik bouncer**. CrowdSec analyses logs, detects bad behaviour and maintains a local blocklist; the bouncer integrates with Traefik to actually block those IPs before they reach the applications.

!!! note
  All values in this page (hostnames, keys, IPs) are examples only.

## Components

The setup consists of two main pieces:

- **CrowdSec agent**  
  - Runs as a container on the same Unraid host.  
  - Ingests logs (e.g. Traefik access logs, system logs).  
  - Applies scenarios from community collections (e.g. HTTP attacks, Traefik‑specific rules).  
  - Maintains a local decision list (ban / captcha / allow) and can optionally share signals with the CrowdSec community.
- **Traefik CrowdSec bouncer**  
  - A small web service that Traefik calls via **ForwardAuth** for each incoming request.  
  - Checks the client IP against the CrowdSec decisions API.  
  - Returns `403 Forbidden` if the IP is banned; otherwise the request continues as normal.

This turns Traefik into a simple but effective application‑level firewall for all HTTP(S) services.

## CrowdSec on Unraid

On Unraid, the CrowdSec container:

- Uses the `crowdsecurity/crowdsec` image
- Is attached to the same custom Docker network as Traefik (e.g. `docknet`)
- Mounts:
  - A data directory for CrowdSec state, e.g. `/mnt/user/appdata/crowdsec/data` → `/var/lib/crowdsec/data`
  - A configuration directory, e.g. `/mnt/user/appdata/crowdsec` → `/etc/crowdsec`
  - One or more log sources, such as Traefik access logs
- Loads relevant collections via an environment variable such as:

  ```text
  COLLECTIONS=crowdsecurity/traefik crowdsecurity/http-cve
  ```

Optionally, the agent can be enrolled in the CrowdSec console using an enrol key, but that detail is not needed for the basic local protection.

## Traefik bouncer

The Traefik bouncer runs as a separate container using the `fbonalair/traefik-crowdsec-bouncer` image.

Key environment variables:

- `CROWDSEC_BOUNCER_API_KEY=<BOUNCER_API_KEY>` – API key created in CrowdSec for this bouncer.
- `CROWDSEC_AGENT_HOST=crowdsec:8080` – how the bouncer reaches the CrowdSec agent within the Docker network.

The bouncer exposes an HTTP endpoint (by default on port 8080 inside the container) that Traefik will call for each request.

## Integrating with Traefik (ForwardAuth)

In Traefik, the bouncer is configured as a **ForwardAuth middleware**. Conceptually, the dynamic configuration looks like this:

```yaml
http:
  middlewares:
    crowdsec-auth:
      forwardAuth:
        address: "http://crowdsec-traefik-bouncer:8080/api/v1/forwardAuth"
        trustForwardHeader: true
        authResponseHeaders:
          - X-Crowdsec-Decision
```

- `address` points to the bouncer container name and port on the Docker network.  
- `trustForwardHeader` allows the bouncer to respect `X-Forwarded-For` when properly set (for example by Cloudflare or another proxy in front).

This middleware can live in the same Traefik dynamic configuration alongside other middlewares like `auth@file` for Authelia.

## Attaching CrowdSec to services

Once the `crowdsec-auth` middleware exists, you can attach it to any Traefik router. For example, on a protected service you might combine Authelia and CrowdSec like this:

```text
traefik.http.routers.app.rule=Host(`app.example.com`)
traefik.http.routers.app.entryPoints=https
traefik.http.routers.app.middlewares=crowdsec-auth,auth@file
```

Order matters: first CrowdSec decides whether the IP is allowed; then Authelia handles user authentication.

This way, obvious attackers or scanners are blocked before they even see a login page, while legitimate users still get the normal SSO experience.

## Log sources

For HTTP‑level protection, the important log source is the **Traefik access log**. In the CrowdSec configuration, a parser/scenario combination for Traefik is enabled (via the `crowdsecurity/traefik` collection). This allows CrowdSec to detect patterns like:

- Repeated 404/401/403 from the same IP
- Known HTTP exploit signatures
- High request rates from a single source

Additional logs (system logs, SSH, etc.) can be added as needed, but are beyond the scope of this networking‑focused page.

## Privacy and documentation

Although CrowdSec processes IP addresses and request metadata, this public documentation:

- Does **not** include real IP addresses, hostnames or API keys
- Uses generic placeholders like `app.example.com`, `<BOUNCER_API_KEY>`, `crowdsec:8080`
- Describes only the architecture and patterns, not live configuration values

This keeps the homelab safer while still sharing the design and lessons learned.
