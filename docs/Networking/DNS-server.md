---
authors:
    - Bernd Verhofstadt
weight: 43
---
# Local DNS server: Blocky

A local DNS resolver can block unwanted domains and provide local name
resolution. Configure the router's DHCP settings to advertise the address of
your DNS server to clients. Choose an unused address on your own LAN; the
addresses in this page are documentation examples only.

## Configuration example

Blocky's configuration is stored in a file such as
`/mnt/user/appdata/blocky/config/config.yml`:

```yaml
upstream:
  default:
    # Replace with a resolver you trust.
    - tcp-tls:dns.example.org:853

startVerifyUpstream: true

customDNS:
  mapping:
    server2.example.org: 192.168.1.24
    app.example.org: 192.168.1.23

ports:
  dns: 53
  tls: 853
  https: 443
  http: 4000

bootstrapDns: tcp+udp:dns.example.org

filtering:
  queryTypes:
    - AAAA
```

Replace sample names, addresses and resolver settings with values for your
network. Use split-horizon records when an internal-only service should resolve
to a private address only for LAN clients.

## Container setup

DNS services usually need direct access to the LAN and commonly use a
dedicated LAN attachment rather than the application `proxy` network. Reserve
a static address outside the DHCP allocation pool, for example
`192.168.1.23`, and verify it is not already in use. Keep the configuration
and logs in persistent host paths, and allow TCP and UDP on the configured
DNS port.
