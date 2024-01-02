---
authors:
    - Bernd Verhofstadt
weight: 31
---
# Selecting the Right Hardware
## A Glimpse into My Unraid Servers
In the realm of home labs, choosing the appropriate hardware lays the foundation for a seamless and efficient exploration. In my setup, I currently operate two Unraid servers, each serving distinct purposes, leveraging the capabilities of diverse hardware configurations.

### HP USSF (Ultra Small Form Factor)

Purpose: This compact powerhouse excels in handling basic infrastructure services within my home lab.
Services Running:
- Reverse Proxy (Traefik): Directing and managing incoming traffic to the appropriate internal servers.
- Local DNS Server: Facilitating seamless redirection for local network users without traversing the internet.
- Unifi Network Application: Managing and optimizing network resources for a cohesive user experience.
- Cloudflared Tunnel: Establishing secure and efficient connections through Cloudflare.
- Storage Considerations: With limited storage options (only one SATA and one M.2 slot), this server focuses primarily on services without requiring extensive data storage.

### Supermicro Desktop

Purpose: Designed for versatility, this Supermicro server hosts an array of self-hosted services, extending beyond basic infrastructure needs.
Additional Services:
- Basic Infrastructure Services: Similar to the USSF, providing essential functionalities.
- Filerun: A self-hosted cloud file server, akin to OwnCloud, for secure data management.
- Authentication Service (Authelia): Ensuring robust security measures for user authentication.
- Home Automation (NodeRed and OpenHab): Creating a smart and interconnected home environment.
- Productivity Services: Including Drawio, Codex, Rallly, Ghostfolio, Crypteogen, fostering a productive and efficient workspace.

The Supermicro server, with its expanded storage options, accommodates a diverse range of services that require more substantial data handling. This strategic allocation allows for optimal resource utilization, ensuring that each server operates efficiently within its designated scope.

As you embark on your hardware selection journey, consider the specific needs of your home lab and tailor your choices accordingly. Whether it's a compact USSF or a versatile Supermicro desktop, the key is aligning your hardware with the services and capabilities you aim to explore in your Unraid home lab environment.