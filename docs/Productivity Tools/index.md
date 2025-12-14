---
authors:
    - Bernd Verhofstadt
weight: 60
---
# Let the fun begin

![AI generated home cloud](../assets/_913a8aff-d975-4b00-9a42-708c790fc388.jpg)

Productivity tools are where the homelab becomes really fun. On top of the core infrastructure (Unraid, Traefik, authentication, storage) I run a collection of self‑hosted apps for documents, tasks, assets and collaboration.

All of these services run as Docker containers on Unraid, are exposed through Traefik, and are protected by Authelia/authentik depending on the use case. Hostnames, IP addresses and credentials in this public documentation are intentionally replaced with generic examples.

## Categories of tools

Instead of documenting every single container, I group them by function.

### Documentation and knowledge

For technical notes, homelab documentation and general knowledge management I use:

- A **documentation site** (this MkDocs project) for structured, version‑controlled docs.  
- A **note‑taking / whiteboard app** (for example AFFiNE or a similar tool) for more free‑form planning and brainstorming.

These tools make it easy to capture how the homelab is wired, which greatly simplifies troubleshooting and future changes.

### Files, PDFs and signing

Document‑centric workflows are handled by a combination of tools:

- A **file management / drive** solution for storing and organising documents.  
- A **PDF toolbox** (for example Stirling‑PDF) for merging, splitting, converting and OCR.  
- An **e‑signature platform** (for example Docuseal) to sign and collect signatures on documents.

All of these live behind the reverse proxy and authentication layer so that personal documents are never exposed directly to the internet.

### Asset and inventory tracking

To keep track of hardware and household items I use:

- An **asset management** tool (for example Homebox) to register devices, locations, serial numbers and purchase information.
- A **data/workspace app** (for example a NocoDB‑like tool) for simple structured lists and tables (e.g. projects, checklists, inventories).

This replaces ad‑hoc spreadsheets with something that is easier to query and extend over time.

### Scheduling and collaboration

For coordinating with others and planning events:

- A **meeting/polling tool** (for example Rallly or similar) to find suitable times.  
- Lightweight dashboards or launchers that collect links to the most used services.

Again, all of these are fronted by Traefik and protected with Authelia/authentik so that only authenticated users can access them.

## Common deployment pattern

Most productivity containers follow the same high‑level pattern:

1. **Run as a Docker container** on Unraid with appdata persisted under `/mnt/user/appdata/<app-name>`.
2. **Attach to the reverse‑proxy network** (for example a custom Docker network used by Traefik).
3. **Expose only internal ports**; no direct exposure to the host network. Traefik handles external HTTPS.
4. **Add Traefik labels** with a hostname such as `app.example.com` and the internal service port.
5. **Protect access** with either:
    - Authelia as forward‑auth, or
    - Direct OpenID Connect integration with authentik as IdP.

This keeps the configuration consistent across apps and makes it easy to spin up or retire tools without re‑thinking the basic architecture each time.

## Future extensions

As new needs appear (budgeting, personal finance dashboards, additional collaboration tools, …) I can add more containers that fit into the same pattern. The important part is that every new service:

- Lives behind the reverse proxy
- Is protected by the authentication stack
- Avoids hard‑coding real hostnames, IP addresses or secrets into documentation

so the overall productivity layer grows in a controlled and secure way.