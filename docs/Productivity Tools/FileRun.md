---
authors:
  - Bernd Verhofstadt
weight: 61
---
# File management with FileRun

For structured file storage and web-based access to documents I use **FileRun**, backed by a dedicated MariaDB database. It provides a familiar file manager interface in the browser with sharing, previews and user permissions.

All examples on this page are sanitised: hostnames, paths and credentials are placeholders.

## Architecture

The FileRun stack consists of:

- A **MariaDB** database container for FileRun metadata (users, shares, file indexes)
- A **FileRun application** container serving the web UI
- Persistent storage for:
  - The FileRun application data
  - The MariaDB data directory
- Traefik handling HTTPS and routing a hostname such as `files.example.com` to the FileRun container

On Unraid, the MariaDB part looks conceptually like:

- Image: `mariadb:11`
- Environment variables:
  - `MARIADB_DATABASE=filerun_db`
  - `MARIADB_USER=filerun_user`
  - `MARIADB_PASSWORD=<STRONG_DB_PASSWORD>`
- Volumes:
  - `/mnt/user/appdata/mariadb-filerun/data:/var/lib/mysql`

The FileRun container then connects to this database using the same credentials.

## Deployment pattern on Unraid

High-level steps to deploy FileRun on Unraid:

1. **Create a MariaDB container** for FileRun:
   - Set `MARIADB_DATABASE`, `MARIADB_USER`, `MARIADB_PASSWORD` to strong values.
   - Map the data directory to an appdata share (preferably on cache for performance).
2. **Deploy the FileRun container**:
   - Point its database settings to the MariaDB container (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`).
   - Mount one or more host folders as FileRun storage roots.
3. **Attach both containers to the reverse proxy network** so Traefik can route to FileRun.
4. **Add Traefik labels** on the FileRun container, for example:

   ```text
   traefik.enable=true
   traefik.http.routers.filerun.rule=Host(`files.example.com`)
   traefik.http.routers.filerun.entryPoints=https
   traefik.http.services.filerun.loadbalancer.server.port=80
   ```

5. **Protect the hostname** with Authelia or authentik so that only authenticated users can access the web UI.

## Usage in the homelab

Typical use cases for FileRun in this setup:

- Centralised storage for PDFs, documents and exports from other services
- Quick web access to important files from any device
- Sharing temporary links with limited lifetime or password protection

FileRun integrates nicely with the rest of the homelab:

- Backups are handled at the storage level (appdata + document shares)
- Access is fronted by the same Traefik/Authelia/authentik stack used elsewhere
- No sensitive internal paths or credentials are ever exposed in the public documentation
