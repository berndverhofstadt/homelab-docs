---
authors:
  - Bernd Verhofstadt
weight: 61
---
# File management with FileRun

For structured file storage and web-based access to documents I use **FileRun**, backed by a dedicated MariaDB database. It provides a familiar file manager interface in the browser with sharing, previews and user permissions.

All examples on this page are sanitised: hostnames, paths and credentials are placeholders.

## Architecture

The FileRun deployment is split into independent development and production
Compose projects. Each environment has its own application, MariaDB database,
Tika processing service, Collabora instance and Elasticsearch service:

- A **FileRun application** container serving the web UI
- A dedicated **MariaDB** database for metadata
- **Tika**, **Collabora** and **Elasticsearch** services for document
  processing, editing and search
- Persistent storage for application data, the database, search index and
  service configuration
- Traefik routing each front door, for example `drive.example.org`,
  `dev-drive.example.org`, `office.example.org` and
  `dev-office.example.org`, to the appropriate environment

Keep the environments isolated: development can be started or stopped without
changing production. Their front doors join `proxy`; app and data services
join only their environment's private networks. The database and search
services must not be shared across environments.

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

For a new installation, high-level steps are:

1. **Create a MariaDB container** for FileRun:
   - Set `MARIADB_DATABASE`, `MARIADB_USER`, `MARIADB_PASSWORD` to strong values.
   - Map the data directory to an appdata share (preferably on cache for performance).
2. **Deploy the FileRun container**:
   - Point its database settings to the MariaDB container (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`).
   - Mount one or more host folders as FileRun storage roots.
3. **Attach the front door to `proxy`** and keep the database on the
   environment's private data network.
4. **Add Traefik labels** to each routed front door, for example:

   ```text
   traefik.enable=true
   traefik.http.routers.filerun.rule=Host(`drive.example.org`)
   traefik.http.routers.filerun.entryPoints=https
   traefik.http.services.filerun.loadbalancer.server.port=80
   ```

5. **Configure the intended access policy** for the application. Authentication
   is service-specific and is not assumed to be provided by a universal
   middleware.

## Usage in the homelab

Typical use cases for FileRun in this setup:

- Centralised storage for PDFs, documents and exports from other services
- Quick web access to important files from any device
- Sharing temporary links with limited lifetime or password protection

FileRun integrates nicely with the rest of the homelab:

- Backups are handled at the storage level (appdata + document shares)
- Access is routed through Traefik; the authentication policy is configured
  for the application
- No sensitive internal paths or credentials are ever exposed in the public documentation
