---
authors:
    - Bernd Verhofstadt
weight: 35
---
# Unraid backup and restore strategy

A reliable backup and restore strategy is essential for any homelab. For my Unraid setup I use an automated configuration export that captures container definitions and core system settings, so I can quickly rebuild the environment on new hardware or after a failure.

This page explains the overall approach without exposing any sensitive data (IP addresses, hostnames, credentials, or internal domains).

## What is backed up

The Unraid backup captures:

- **System configuration** – array layout, shares, Docker and VM settings
- **Container templates** – Unraid XML templates for each container (all environment variables, port mappings, volumes, etc.)
- **Docker Compose fallback** – a generated `docker-compose.yml` that mirrors the running containers
- **Helper scripts and metadata** – a small restore script and a README with recovery instructions

Application data (databases, appdata folders, media, etc.) is backed up separately using dedicated tools and is not covered in detail on this page.

## Backup tooling

The configuration export is automated using a small container that regularly:

- Reads the current Unraid Docker configuration via the Docker API
- Serialises container definitions into:
  - Unraid XML templates (for the native Unraid Docker interface)
  - A consolidated `unraid-config.json` with system/container metadata
  - A `docker-compose.yml` as an emergency fallback
- Writes the results to a designated backup share on the array

Backups are scheduled to run **weekly**. The output is then included in the regular backup of the homelab data (for example via rclone or another backup tool).

## Restore methods

There are two main ways to restore containers from this backup.

### Method 1: Native Unraid templates (recommended)

This method keeps you fully within the Unraid GUI and Docker management:

1. Install a fresh Unraid system and assign disks to the array.
2. Restore the Unraid flash drive backup if available.
3. Copy the exported container templates to the Unraid USB under:
   - `/boot/config/plugins/dockerMan/templates-user`
4. In the Unraid **Docker** tab, choose **Add Container** and pick from the restored templates.
5. For each container:
   - Verify volume mappings
   - Point appdata volumes to the restored locations
   - Adjust any environment variables that should not be restored verbatim (API keys, passwords, …).
6. Start the containers and verify that applications connect correctly to their data.

This method is preferred because it keeps all container definitions in Unraid’s native format and UI.

### Method 2: Docker Compose (emergency fallback)

If the Unraid GUI is not available or you need a very quick recreation of containers, you can use the generated `docker-compose.yml` as a secondary option.

1. Copy the `docker-compose.yml` from the backup to a safe location on the Unraid server.
2. From the Unraid console or SSH, navigate to that folder.
3. Start the stack with:

   ```bash
   docker compose up -d
   ```

4. Once the containers are running, you can fine‑tune volumes and environment variables and, if desired, later migrate definitions back into Unraid templates.

> **Note**  
> Docker Compose should be seen as a fallback. For day‑to‑day operation and long‑term maintainability, I use the Unraid Docker tab with XML templates.

## Separation of configuration and secrets

Because this documentation is public, and to keep the backups safe:

- Real **hostnames**, **domains**, **IP addresses**, and **credentials** are **never** committed to this repository.
- In configuration exports, secrets are either masked or replaced with placeholders before being documented.
- Public examples in this documentation use generic values such as `example.com`, `192.168.1.10`, `<TOKEN>`, …

When restoring from a real backup, always review environment variables and configuration files and re‑enter:

- Passwords and API keys
- SMTP credentials
- External URLs and callback endpoints

## Sanitization notes for this repository

Across all pages in this documentation, concrete identifiers from the private homelab are intentionally replaced with placeholders. Some conventions you will see:

- Domains and hostnames: `example.com`, `app.example.com`, `id.example.com`, `unraid.example.com`
- Internal IPs: `192.168.x.x`, `10.x.x.x` (or specific examples like `192.168.1.10`)
- Secrets and credentials: `<TOKEN>`, `<BOUNCER_API_KEY>`, `<REDIS_PASSWORD>`, `<SMTP_PASSWORD>`, `<cloudflared-token>`, `<STRONG_DB_PASSWORD>`

If you copy a snippet from these docs into your own configuration, make sure to:

- Replace every placeholder with a real value appropriate for your environment.
- Avoid committing your own real hostnames, IPs or secrets back into this public repository.

## Testing the restore process

A backup is only useful if it can be restored. Periodically:

- Spin up a test Unraid instance (for example on spare hardware or a VM)
- Follow **Method 1** using the exported templates
- Validate that key services (reverse proxy, authentication, DNS, storage, automation, …) come back online

This gives confidence that the documented process works and that no hidden dependencies (like hard‑coded IPs or hostnames) remain.
