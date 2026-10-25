---
authors:
    - Bernd Verhofstadt
weight: 41
---
# Docker Compose and network segmentation

The homelab uses Docker Compose as the source of truth for application stacks.
Each stack has its own Compose file and a short README describing the setup,
data that must be preserved, and deployment checks. Compose makes it possible
to define dependencies, health checks, restart policies and network membership
alongside each service.

## Networks on each host

Docker networks are local to one host. The primary and secondary Unraid hosts
therefore define independent networks, even when those networks have the same
names. A container on one host cannot join a Docker network on the other host.

Each host follows the same network roles:

| Network | Purpose |
| --- | --- |
| `proxy` | Traefik and containers that need their own HTTP(S) hostname. A stack may have multiple front doors. |
| `shared-services` | Services intentionally consumed by multiple application stacks. MQTT is the current example; consumers also remain on their own stack networks. |
| `traefik-support` | Traefik's private helpers, such as CrowdSec, its bouncer, the Docker socket proxy and Cloudflared. These services do not need public hostnames. |
| Per-stack networks | Isolate each application's services and backends from unrelated stacks. |

The earlier flat `docknet` design let unrelated containers reach one another's
ports. The segmented design limits shared connectivity to the cases that need
it. Do not attach a container to a shared network merely to make it reachable
from another stack.

## Application and data tiers

Stacks with a separate database, cache, search engine or object store use two
private networks:

- `<stack>-data` is `internal: true` and contains only backend services.
- `<stack>-app` is `internal: true` by default. Application containers join
  both tiers so they can reach their own backend; only an application that
  demonstrably requires outbound access should receive an egress exception.

This keeps backend services off networks with internet egress. A service with
no separate backend, such as a single-container utility, does not need an
empty app/data split; it joins `proxy` only when it has a routed hostname.
Databases and caches are dedicated to their application rather than shared
between unrelated stacks.

Front-door containers join `proxy`; private backends do not. For a multi-homed
front door, configure Traefik to use the `proxy` network explicitly. Do not
publish application ports directly on the host when Traefik is the intended
entry point.

## Host roles and exposure

The primary host runs the internet-facing services and their Traefik edge.
The home-automation design places openHAB, its MQTT broker and the device
bridges together on a separate host. That host has its own networks and is
designed for LAN-only access: it is not connected to the public tunnel, and
internal DNS can resolve names such as `openhab.internal.example.org` to the
LAN service. DNS-01 can provide trusted TLS certificates without opening an
inbound internet route.

Treat this as an exposure policy, not an assumption based on a hostname.
Verify tunnel ingress, DNS and reachability from both inside and outside the
LAN before describing a service as LAN-only.

## Examples from the Compose layout

- Cryptgeon has a dedicated Redis backend on its own internal data network.
- Immich has its own PostgreSQL and Redis backends.
- FileRun development and production are independent Compose deployments. Each
  has its own database, search and processing services, so either environment
  can be operated without changing the other.
- Single-container tools use the proxy network without a separate backend
  tier. Stateful applications persist their data separately.

## Deployment and verification

Bring up a stack only after its required external networks exist. Use
health-based dependencies when an application needs a backend to be ready
before startup, and choose restart behavior appropriate to each service.
Avoid applying changes to unrelated stacks as part of a cutover.

For each change, verify:

1. **Startup and health:** Compose configuration is valid, services become
   healthy, and dependencies are ready before dependent applications start.
2. **Routing:** each intended front door works through Traefik, with no
   application port unintentionally published on the host.
3. **Isolation:** backends are reachable only from their intended stack, and
   data-tier containers have no internet egress.
4. **Exposure:** services intended to be LAN-only are reachable on the LAN and
   unavailable from outside it.

For stateful services, preserve the existing data and confirm backup coverage
before changing mounts, databases or images. See the
[backup and restore strategy](../Hardware-&-OS/Unraid-Backup-and-Restore.md).

## Data changes and image updates

Treat a volume-path change as a storage cutover, not an application migration.
Confirm the intended pool is mounted and contains the complete data on that
host; stop writers, take a recoverable backup, change one workload at a time,
and verify the application and backup afterward. Direct pool paths such as
`/mnt/cache/appdata/<stack>` are targeted exceptions, not a blanket
replacement for `/mnt/user` paths. Do not copy between those paths without
checking their relationship; they may refer to the same underlying files.
Do not combine a mount change with an image upgrade or database rebuild.

The update policy is based on state, not image tags: stateless services with
no durable data or dependents may be eligible for automated updates, stateful
services need a human-reviewed update, and FileRun remains manually reviewed.
The automation for this policy is planned separately and should not be
described as deployed until it is operating and verified.

All hostnames, addresses and credentials in these docs are illustrative.
Replace `example.org`, `192.168.1.23` and angle-bracket placeholders with
values appropriate to your own network; never publish live credentials.
