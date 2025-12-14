---
authors:
    - Bernd Verhofstadt
weight: 90
---
# Maintaining these docs

This section is a short reminder for future you (or contributors) on how to work with the VCloud Homelab Docs.

## Running the docs locally

1. Create and activate a Python virtual environment (optional but recommended).
2. Install dependencies:

   ```bash
   pip install mkdocs mkdocs-material mkdocs-nav-weight
   ```

3. From the repository root, run:

   ```bash
   mkdocs serve
   ```

4. Open the URL shown in the terminal (by default `http://127.0.0.1:8000`).

## Structure and navigation

- All content lives under the `docs/` folder.
- Section index pages:
  - `Hardware-&-OS/index.md`
  - `Networking/index.md`
  - `Authentication/index.md`
  - `Home-Automation/index.md`
  - `Productivity Tools/index.md`
- New pages should be added in the appropriate section and linked from the relevant index page.

MkDocs navigation is driven primarily by the directory structure; the `weight` field in the front matter is used (via `mkdocs-nav-weight`) to order pages within a section.

## Adding a new page

1. Create a new Markdown file under `docs/` in the right section.
2. Add a minimal front matter block, for example:

   ```yaml
   ---
   authors:
       - Bernd Verhofstadt
   weight: 50
   ---
   ```

3. Link to the new page from the relevant index page using a relative link.
4. Run `mkdocs serve` and verify that navigation and links work as expected.

## Placeholder and sanitization rules

Because this repository is public and based on a private homelab:

- Never commit real domains, public IPs, passwords, API keys or tokens.
- Use placeholders in all examples, for example:
  - Domains/hostnames: `example.com`, `app.example.com`, `id.example.com`
  - Internal IPs: `192.168.x.x`, `10.x.x.x`
  - Secrets: `<TOKEN>`, `<BOUNCER_API_KEY>`, `<SMTP_PASSWORD>`, `<cloudflared-token>`

The page `Hardware-&-OS/Unraid-Backup-and-Restore.md` contains a more detailed description of these conventions.

## CI and GitHub Pages

A GitHub Actions workflow (`.github/workflows/mkdocs-pages.yml`) builds the site with `mkdocs build --strict` on each push to `main` and deploys it to GitHub Pages.

If a build fails:

- Run `mkdocs build --strict` locally to see the error.
- Fix broken links, missing files or front matter issues.
- Commit and push again.

## Updating dependencies

From time to time, update MkDocs and plugins:

```bash
pip install --upgrade mkdocs mkdocs-material mkdocs-nav-weight
```

Re-run `mkdocs build --strict` to ensure everything still works.
