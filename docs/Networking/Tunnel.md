---
authors:
    - Bernd Verhofstadt
weight: 41
---
# Cloudflared tunnel

Cloudflared creates an outbound connection from a private network to
Cloudflare. It can publish selected hostnames without opening inbound router
ports. A tunnel does not make a service private by itself: each configured
public hostname may be reachable by anyone unless access controls are added.

## Configure a tunnel

Create a tunnel in the Cloudflare dashboard and store its token as a secret.
For a container deployment, pass the token through a protected environment
file or secret store, not a committed Compose file:

```sh
tunnel --no-autoupdate run --token <CLOUDFLARED_TOKEN>
```

Place Cloudflared on the host's `traefik-support` network and configure the
tunnel to forward selected hostnames to Traefik. Do not attach it to each
application network. The tunnel and Traefik should be able to reach one
another without giving the tunnel access to application backends.

An example public hostname is `app.example.org`, forwarded to the local
Traefik HTTPS entry point. Use a generic example when documenting origins, for
example `https://192.168.1.23:443`; replace it with the address or service
name appropriate to your own setup.

## Public and LAN-only services

Only add a hostname to tunnel ingress when it is intended to be public. A
service routed by Traefik is not automatically public; public reachability
depends on DNS, tunnel ingress and other network paths.

The home-automation host is designed to remain LAN-only and must not be added
to the public tunnel. It can use split-horizon DNS and a DNS-01 certificate
for a hostname such as `openhab.internal.example.org`. DNS-01 validates
domain ownership without creating an inbound route to the service. Verify
LAN-only behavior from both an on-LAN and off-LAN client.

## Safety checklist

- Keep the tunnel token secret and rotate it if exposed.
- Publish only the hostnames that need public access.
- Apply an application-level login or an explicit access policy where needed.
- Keep internal-only services off public tunnel ingress.
- Test actual external reachability; a hostname naming convention is not an
  access control.
