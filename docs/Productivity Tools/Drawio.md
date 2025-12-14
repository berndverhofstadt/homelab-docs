---
authors:
    - Bernd Verhofstadt
weight: 63
---
# Diagramming with draw.io

For architecture diagrams, network layouts and homelab sketches I self‑host the **draw.io** web application.

## Role in the homelab

draw.io is used to document:

- Network topologies (VLANs, routers, firewalls, tunnels)
- Service layouts (reverse proxy, authentication, storage, automation)
- Hardware setups (rack layout, cabling)

Storing these diagrams alongside other documentation makes it much easier to reason about the environment and plan changes.

## Deployment pattern

The container is based on the official `jgraph/drawio` image and runs as a stateless web app behind Traefik.

Key characteristics:

- No internal database; diagrams are typically saved to network shares or downloaded locally
- Container listens on an internal HTTP port (e.g. 8080)
- Traefik exposes it via HTTPS on a hostname like `draw.example.com`
- Access is protected using Authelia via the `auth@file` middleware

Example Traefik labels on the draw.io container:

```text
traefik.enable=true
traefik.http.routers.drawio.rule=Host(`draw.example.com`)
traefik.http.routers.drawio.entryPoints=https
traefik.http.routers.drawio.middlewares=auth@file
```

## Storage of diagrams

There are multiple ways to store diagrams when self‑hosting draw.io:

- Save `.drawio` files to a documents share managed by FileRun or another file service
- Export to PNG/SVG/PDF and commit them into version‑controlled documentation (like this repo)

In all cases, the actual storage locations and paths are kept private and are not exposed in this public documentation.
