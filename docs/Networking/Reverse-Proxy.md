---
authors:
    - Bernd Verhofstadt
weight: 42
---
# Reverse Proxy: Traefik
In my homelab setup, all incoming 80 and 443 traffic must go through the reverse proxy. Traefik makes sure that the connection is made on the docker network layer and that all requests are moved from HTTP (port 80) to HTTPS (port 443), next to that, Traefik handles the SSL certificates for the domain. 

To make use of these ports, the default ports from Unraid must be modified. Ports can be modified in Unraid under Settings > Management Access > HTTP port & HTTPS port. Note that after this change you must use the designated port, e.g. http://unraid.local:81.

Excellent tutorials where I started with:

1. [Unraid - Traefik v2.6+ (ibracorp.io)](https://docs.ibracorp.io/traefik/master/unraid)
1. [Proxying Your First App - Traefik v2.6+ (ibracorp.io)](https://docs.ibracorp.io/traefik/master/unraid/proxying-your-first-app)

## Dockersocket
Install the container dockersocket. This container makes sure that Traefik will be able to read the labels created on other containers. Make sure the container has the custom created network.

Property      | Value | Details | Description
---|---|---|---
Network Type: | Custom: dockersock    | |
Containers: | 1 | Container Variable: CONTAINERS | Allow access to running containers
docker.sock: | /var/run/docker.sock | Container Path: /var/run/docker.sock

## Traefik configuration
Traefik provides two ways of hostname mappings, automatically via container labels or manual via a config file. I have a mix of both as some services are not running in a container (e.g. Unraid dashboard)

Create a file called `traefik.yml` which must be uploaded in `/mnt/user/appdata/traefik/`

```yaml
http:
  ## EXTERNAL ROUTING - Only use if you want to proxy something manually ##
  routers:
    # UNRAID routing - Remove if not used
    unraid:
      entryPoints:
        - https
      rule: "Host(`unraid.example.com`)"
      service: unraid
  ## SERVICES ##
  services:
    # UNRAID service - Remove if not used
    unraid:
      loadBalancer:
        servers:
          - url: "http://192.168.1.10:81/"
  ## MIDDLEWARES ##
  middlewares:
    # Only Allow Local networks
    local-ipwhitelist:
      ipWhiteList:
        sourceRange:
          - 127.0.0.1/32 # localhost
          - 172.16.0.0/12 # LAN Subnet

    # Security headers
    securityHeaders:
      headers:
        customResponseHeaders:
          X-Robots-Tag: "none,noarchive,nosnippet,notranslate,noimageindex"
          X-Forwarded-Proto: "https"
          server: ""
        customRequestHeaders:
          X-Forwarded-Proto: "https"
        sslProxyHeaders:
          X-Forwarded-Proto: "https"
        referrerPolicy: "same-origin"
        hostsProxyHeaders:
          - "X-Forwarded-Host"
        contentTypeNosniff: true
        browserXssFilter: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsSeconds: 63072000
        stsPreload: true

# Only use secure ciphers - https://ssl-config.mozilla.org/#server=traefik&version=2.6.0&config=intermediate&guideline=5.6              
tls:
  options:
    default:
      minVersion: VersionTLS12
      cipherSuites:
        - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
        - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
        - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
        - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
        - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
        - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305

```

## Setup Traefik container

### Get Cloudflare API token
The only additional setting you'll need to get is the one from Cloudflare to verify the SSL certificates

1. Open Cloudflare > My Profile > API Tokens 
2. Create a new token > Edit DNS zone > use template.
3. Create multiple permissions:
    - Zone - Zone Settings - Read
    - Zone - Zone - Read
    - Zone - DNS - Edit
4. Add Zone Resource
    - Include - All Zones

After creation, copy the token and use it in your Traefik configuration.

### Traefik configuration properties
Property | Value | Details | Description
---|---|---|---
Network Type:  | Custom: dockernet |  | 
Config Folder:  | /mnt/user/appdata/traefik  | Container Path: /etc/traefik | Appdata Location
Docker Socket:  | /var/run/docker.sock  | Container Path: /var/run/docker.sock | The default is /var/run/docker.sock
Https Port:  | 443 | Container Port: 443 | HTTPS Port
Http Port: | 80 | Container Port: 80 | HTTP Port
Web UI Port:  | 8183 | Container Port: 8080  | Dashboard WebUI Port
Cloudflare API Token:  | < token > | Container Variable: CF_DNS_API_TOKEN  | Cloudflare DNS API Token
Traefik Dashboard Subdomain:  | Host(\`traefik.example.com\`)  | Container Label: traefik.http.routers.api.rule | Traefik dashboard URL 
Traefik Entrypoint:  | https | Container Label: traefik.http.routers.api.entryPoints  | Traefik Dashboard Entrypoint
Traefik API:  | api@internal | Container Label: traefik.http.routers.api.service | Routing Traefik to its API Dashboard  
Enable Traefik (Dashboard):  | true | Container Label: traefik.enable  | Enable/Disable Traefik Dashboard
Docker Socket Proxy:  | dockersocket | Container Variable: DOCKER_HOST | Equal to the name of the dockersocket container   

For protection against malicious IPs and HTTP attacks, I integrate Traefik with CrowdSec and its Traefik bouncer. The high‑level design and configuration pattern are described on the dedicated CrowdSec networking page.