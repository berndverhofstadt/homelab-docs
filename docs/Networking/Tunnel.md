---
authors:
    - Bernd Verhofstadt
weight: 41
---
# Cloudflared tunnel

## Cloudflare account setup
To use Cloudflare’s free service for small setups, you need to create an account first. You can find many helpful videos online, or read the Cloudflare documentation [here](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/).

## Domain name setup
To make use of this service, you need to configure or buy a domain at [Cloudflare](https://developers.cloudflare.com/registrar/account-options/domain-management).

## Configure token
After you have an account and a tunnel token from the Zero Trust dashboard on Cloudflare, go to Unraid and install the Cloudflared Tunnel container from the community apps. Don’t forget to switch to advanced view. Then, in the **Post Arguments** fields, type this and replace `<cloudflared-token>` with your actual token:

```bash
tunnel --no-autoupdate run --token <cloudflared-token>
```

Finally, make sure to select the created custom docker network.


### Configure public hostnames
You need to go to the Access section and open Tunnel on the Cloudflare Zero Trust dashboard. Then, choose the tunnel and click on configure to see the token. Go to the Public hostnames tab and add a new one. This tunnel lets you access any local resources that the host can access. You can test it by creating a public hostname for a service on your network (for example, router config at 192.168.1.1). But remember, the public domain is open to everyone by default. So, I suggest you delete the public hostname when you are done testing. I have my final configuration pointing to my reverse proxy, which handles all my local traffic and certificates. The service field is for the reverse proxy’s hostnames. If your reverse proxy is not in the docker setup (for example, another Unraid server), you need to add the IP address.

### Example for accessing Unraid dashboard 
More info at [Traefik](https://docs.verhofstadt.cloud/networking/traefik) on how to correct configure reverse proxy

Public hostname: `unraid.mydomain.com`  
Services: `https://traefik:443`  

*Additional application settings*  
Origin Server Name: `unraid.mydomain.com`  
HTTP2 connection: `enabled`  

### Example for accessing the OpenSpeedTest container
Public hostname: `openspeedtest.mydomain.com`  
Services: `https://traefik:443`

*Additional application settings*  
Origin Server Name: `openspeedtest.mydomain.com`  
HTTP2 connection: `enabled`  