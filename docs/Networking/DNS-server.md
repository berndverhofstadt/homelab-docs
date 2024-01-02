---
authors:
    - Bernd Verhofstadt
weight: 43
---
# Local DNS server: Blocky
Multiple solution are available which act in most cases as an add blocker solution. PiHole is working great but for this case overkill. Let's start with the beginning, why do I use a local DNS server? By default, most setups use a public DNS server which only has records with public IP addresses. By installing your own DNS server, you must make sure that the DHCP server (in most cases build in on your local router), is set to the private IP given to blocky.

## Configuration of blocky
Before you install blocky, a configuration file must be created. All instructions are available in the [documentation](https://0xerr0r.github.io/blocky/configuration/) of Blocky. The config file is named as config.yml and must be uploaded in:  
```/mnt/user/appdata/blocky/config/```

An example file can look like the following:

```yaml
upstream:
  # these external DNS resolvers will be used. Blocky picks 2 random resolvers from the list for each query
  # format for resolver: [net:]host:[port][/path]. net could be empty (default, shortcut for tcp+udp), tcp+udp, tcp, udp, tcp-tls or https (DoH). If port is empty, default port will be used (53 for udp and tcp, 853 for tcp-tls, 443 for https (Doh))
  # this configuration is mandatory, please define at least one external DNS resolver
  default:
    # DNS tcp udp
    - tcp-tls:193.110.81.9:853
    - tcp-tls:185.253.5.9:853

# optional: If true, blocky will fail to start unless at least one upstream server per group is reachable. Default: false
startVerifyUpstream: true

# optional: definition, which DNS resolver(s) should be used for queries to the domain (with all sub-domains). Multiple resolvers must be separated by a comma
# Example: Query client.fritz.box will ask DNS server 192.168.178.1. This is necessary for local network, to resolve clients by host name
customDNS:
  mapping:
    server2.example.com: 192.168.1.11
    example.com: 192.168.1.10

# optional: ports configuration
ports:
  # optional: DNS listener port(s) and bind ip address(es), default 53 (UDP and TCP). Example: 53, :53, "127.0.0.1:5353,[::1]:5353"
  dns: 53
  # optional: Port(s) and bind ip address(es) for DoT (DNS-over-TLS) listener. Example: 853, 127.0.0.1:853
  tls: 853
  # optional: Port(s) and optional bind ip address(es) to serve HTTPS used for prometheus metrics, pprof, REST API, DoH... If you wish to specify a specific IP, you can do so such as 192.168.0.1:443. Example: 443, :443, 127.0.0.1:443,[::1]:443
  https: 443
  # optional: Port(s) and optional bind ip address(es) to serve HTTP used for prometheus metrics, pprof, REST API, DoH... If you wish to specify a specific IP, you can do so such as 192.168.0.1:4000. Example: 4000, :4000, 127.0.0.1:4000,[::1]:4000
  http: 4000

bootstrapDns: tcp+udp:193.110.81.9

filtering:
  queryTypes:
    - AAAA
```

## Setup blocky
The following properties are required to use during the container setup.

Property | Value | Details | Description
---|---|---|---
Network Type:  | Custom: br0  | | Only the default bridge is allowed, the created docker network cannot be used. 
Fixed IP address  | 192.168.1.5  | | Make sure you use an available IP address, preferable outside the DHCP range. 
Config File location:  | /app/config/config.yml  | Container Variable: BLOCKY_CONFIG_FILE | Location of the config file in the container 
Config Dir:  | /mnt/user/appdata/blocky/config  | Container Path: /app/config |
Logs Dir:  | /mnt/user/appdata/blocky/logs  | Container Path: /logs |
DNS Port TCP:  | 53 | Container Port: 53 (TCP) |
DNS Port UDP  | 53 | Container Port: 53 (UDP) |