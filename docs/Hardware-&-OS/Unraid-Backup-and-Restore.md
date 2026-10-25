---
authors:
    - Bernd Verhofstadt
weight: 35
---
# Unraid backup and restore strategy

A reliable backup and restore strategy is essential for any homelab. Compose
files are the source of truth for Compose-managed services; persistent
application data and host configuration need separate, verifiable backup
coverage.

## What to back up

Keep recoverable copies of:

- **System configuration** – array layout, shares, Docker and VM settings.
- **Compose definitions** – reviewed files used to recreate each stack.
- **Persistent application state** – databases, appdata, uploaded files,
  indexes and any configuration that cannot be regenerated.
- **Recovery instructions and scripts** – enough detail to restore services
  without relying on undocumented steps.

Back up every container that stores valuable state. Stateless tools and
transient processing services may not need data backups, but their Compose
definitions remain part of the system configuration. Keep an independent
copy outside the source host or pool.

## Backup procedure

Use the existing backup tooling for persistent configuration and data, and
verify its coverage whenever a stack or mount changes. Database backups must
be consistent: stop writers before filesystem copies or use a database-native
backup method. Keep recovery copies outside the source directory and verify
that the restore procedure can read them.

Do not assume that a path is on a particular pool because of its name. Check
the actual mount, share settings and data placement on each host before
changing a stateful volume. Do not copy between merged-user and direct-pool
paths without understanding whether they point to the same underlying files.
Treat direct-pool paths such as `/mnt/cache/appdata/<stack>` as targeted
exceptions, verify the complete data is already on the intended pool, and
change one workload at a time without also upgrading its image or rebuilding
its database.

## Restore Compose-managed stacks

1. Restore the host configuration and reviewed Compose files.
2. Restore persistent data to verified host paths. Restore environment files
   and other secrets from protected backups; do not put them in public source
   control.
3. Confirm required external Docker networks exist and render the Compose
   file:

   ```bash
   docker compose -f compose/primary/example-stack/docker-compose.yml config --quiet
   ```

4. Start one stack at a time, following its README and dependency order:

   ```bash
   docker compose -f compose/primary/example-stack/docker-compose.yml up -d
   ```

5. Confirm service health, routing, access policy and application data before
   restoring another stack.

## Keep public documentation generic

Real hostnames, domains, IP addresses, email addresses, passwords, API keys
and tokens must never be committed to this repository. Use generic values such
as `example.org`, `app.example.org`, `192.168.1.23`, and `<TOKEN>`. Use
reserved example email addresses such as `user@example.org`.

When restoring from a real backup, review environment variables and
configuration files and re-enter or verify passwords, API keys, SMTP
credentials, external URLs and callback endpoints. Do not copy a live
configuration into public documentation without removing real identifiers
and secrets.

## Test restores

A backup is useful only if it can be restored. Periodically restore a
representative stack in a test environment, confirm that its data and
configuration are present, and verify that key services (reverse proxy, DNS,
storage and automation) work without depending on undocumented host values.
