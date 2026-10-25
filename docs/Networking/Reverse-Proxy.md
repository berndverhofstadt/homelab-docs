---
authors:
    - Bernd Verhofstadt
weight: 42
---
# Reverse proxy: Traefik

Traefik is the HTTP(S) entry point for applications that have a routed
hostname. It terminates TLS, selects a backend from Docker labels or dynamic
configuration, and can apply middleware such as CrowdSec or an authentication
provider where appropriate.

## Network placement

Traefik and routed application containers join the host's `proxy` network.
Containers without a hostname do not join `proxy` just for convenience.
Applications with a database or cache also use their own private app/data
networks; their backend services are not attached to `proxy`.

The Docker provider reads container metadata through a Docker socket proxy.
Use the proxy rather than mounting the raw Docker socket into Traefik. Private
edge helpers, including the socket proxy, CrowdSec and its bouncer, use the
host's `traefik-support` network. See the
[network segmentation guide](Docker-Network-Segmentation.md).

## Routing example

Use an explicit router rule and the actual internal service port. Do not
publish the application port on the Unraid host when Traefik is the entry
point.

```yaml
services:
  example-app:
    image: example/image:1.0
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.example-app.rule=Host(`app.example.org`)"
      - "traefik.http.routers.example-app.entrypoints=https"
      - "traefik.http.services.example-app.loadbalancer.server.port=8080"

networks:
  proxy:
    external: true
```

The `traefik.docker.network` label is important when a routed application is
attached to more than one network: it tells Traefik which network to use for
the backend connection. Single-network front doors do not need that
disambiguation label.

## TLS and dashboard

Traefik can obtain certificates using a DNS-01 challenge with a DNS provider.
Store API tokens in a protected environment file or secret manager, not in
Compose files or public documentation. Keep the dashboard private and protect
it with an explicit access policy; do not expose it merely because Traefik
routes application traffic.

If Unraid's management interface uses a non-default port, use a generic
address such as `http://192.168.1.23:81` in examples and restrict access to the
LAN. Do not copy a live host address into public configuration.

## Authentication and middleware

Authentication is configured per application. Some applications use their
own accounts or native identity integration; others may use a Traefik
forward-auth middleware. Do not assume every routed service is protected by
the same provider, and do not add a middleware globally without checking
whether it is appropriate for each route.

CrowdSec and its Traefik bouncer provide request filtering independently of
user authentication. See [Protecting Traefik with CrowdSec](CrowdSec.md).
