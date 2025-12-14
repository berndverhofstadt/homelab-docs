---
authors:
    - Bernd Verhofstadt
weight: 64
---
# Ephemeral secrets with cryptgeon

For sharing sensitive notes or small files that should only be accessible once (or for a limited time), I use **cryptgeon**. It provides client‑side encryption and ephemeral storage so that the server never sees plaintext and does not persist data to disk.

## How cryptgeon works

At a high level:

- The browser encrypts the content client‑side before it is sent to the server.
- The server stores only the encrypted blob and some metadata in an in‑memory store (e.g. Redis).
- A unique link is generated; when opened, the content is decrypted client‑side.
- Notes can be configured to expire after a number of views or a time limit, after which they are deleted from memory.

This makes it ideal for sharing one‑off secrets (temporary passwords, tokens, links) without leaving a long‑lived trace.

## Deployment pattern

The container uses the `cupcakearmy/cryptgeon` image and has:

- A dependency on a Redis instance for in‑memory storage
- Environment variables to point to Redis and adjust the maximum note size
- No local data volume (everything is in memory)

In front of cryptgeon:

- Traefik terminates HTTPS and routes a hostname like `secret.example.com` to the container.
- Authentication can be optional depending on your threat model; I keep it behind my usual authentication stack.

Example environment and labels (sanitised):

```text
REDIS=redis://:<REDIS_PASSWORD>@redis/
SIZE_LIMIT=250MiB

traefik.enable=true
traefik.http.routers.cryptgeon.rule=Host(`secret.example.com`)
traefik.http.routers.cryptgeon.entryPoints=https
```

## Usage guidelines

When using cryptgeon in this homelab:

- Only short‑lived secrets are shared; long‑term passwords are stored in a proper password manager.
- Links are treated as sensitive: once a secret has been used, the note is deleted and the link becomes invalid.
- The public documentation never exposes the real hostname, Redis password or any live cryptgeon links.
