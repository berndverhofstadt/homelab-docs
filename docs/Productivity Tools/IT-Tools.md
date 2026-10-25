---
authors:
    - Bernd Verhofstadt
weight: 62
---
# Developer utilities with IT Tools

For everyday development and troubleshooting tasks I run **IT Tools**, a web app that bundles many small utilities into a single interface (encoders/decoders, formatters, converters, generators, analyzers, …).

## Why host IT Tools yourself?

Many of these utilities exist as random websites on the internet, but self‑hosting them has a few advantages:

- No potentially sensitive snippets leave your network (tokens, payloads, configs)
- Consistent, ad‑free interface
- Integrated into the same authentication and HTTPS setup as the rest of the homelab

## Deployment pattern

The container is very simple:

- Image: `corentinth/it-tools`
- Exposes HTTP on port 80 inside the container
- No database or external dependencies

In Compose, this stateless container:

- Joins the `proxy` Docker network
- Exposed via Traefik with a hostname such as `tools.example.org`
- Uses only the access middleware intended for this application; do not assume
  that a shared authentication layer applies to every route

Example Traefik labels on the IT Tools container:

```text
traefik.enable=true
traefik.http.routers.it-tools.rule=Host(`tools.example.org`)
traefik.http.routers.it-tools.entryPoints=https
```

## Use cases

Common things I use IT Tools for:

- Inspecting and formatting JSON, YAML and XML
- URL/base64/JWT encoding and decoding
- Hashing and checksum calculations
- Small text transformations and generators

Treat it as a convenience utility, not a secure place to process secrets
unless its access policy and the tool's behavior have been reviewed.
