---
authors:
    - Bernd Verhofstadt
weight: 60
---
# Self-hosted productivity tools

This section describes application stacks managed with Docker Compose on
Unraid. Services with a routed hostname use Traefik; a hostname alone does
not mean a service is publicly reachable. Access controls are configured per
application rather than assumed to be uniform. All hostnames and addresses in
examples are generic.

## Files and media

- [FileRun](FileRun.md) provides web-based file management. Development and
  production use separate databases and separate processing/search services.
- Immich is the photo library, with dedicated PostgreSQL and Redis backends.
- Stirling-PDF provides document conversion and OCR. Its example deployment
  is LAN-only; do not add its hostname to public tunnel ingress.

## Inventory, lists and scheduling

- Homebox stores an inventory and attachments using persistent embedded
  storage.
- NocoDB provides structured lists using persistent embedded storage.
- Rallly is a planned polling stack. Its deployment is deferred until it is
  deployed and verified; its planned design uses dedicated PostgreSQL and
  self-hosted object storage.

Some applications in this group are deliberately rebuilt with fresh
configuration rather than migrated from unmanaged containers. That is a
specific data-retention decision for those applications, not the default for
stateful services; preserve valuable data when migrating.

## Developer utilities and ephemeral secrets

- [IT Tools](IT-Tools.md) is a stateless collection of developer utilities.
- [cryptgeon](Cryptgeon.md) stores ephemeral encrypted notes in a dedicated
  Redis instance.
- A QR-generator API is a stateless endpoint for generating QR codes.

## Deployment principles

1. Routed containers join the host's `proxy` network; backends remain on their
   stack's private networks.
2. Do not publish host ports when traffic should pass through Traefik.
3. Persist and back up application data that cannot be recreated. Stateless
   tools and transient processing data may not need backup.
4. Use application login, identity integration or proxy authentication only
   where intended for that service. Do not assume every app has the same
   protection.
5. Keep secrets out of Compose files and public documentation.

For the network layout and validation steps, see
[Docker Compose and network segmentation](../Networking/Docker-Network-Segmentation.md).
