# About This Site

## Stack

| Tool | Purpose |
|---|---|
| [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) | Static site generator + theme |
| [GitHub Pages](https://pages.github.com/) | Free public hosting |
| [GitHub Actions](https://github.com/features/actions) | Auto-deploy on push to main |
| VS Code | Editor |

## Repo

[github.com/Mystikal17/HomeLab-Docs](https://github.com/Mystikal17/HomeLab-Docs)

## Key Commands

```bash
pip install mkdocs-material   # install
mkdocs serve                  # local preview at localhost:8000
mkdocs gh-deploy --force      # manual deploy

# Standard workflow
git add .
git commit -m "description"
git push                      # triggers GitHub Actions auto-deploy
```

## GitHub Actions Deploy

`.github/workflows/deploy.yml` automatically builds and deploys on every push to `main`.

```yaml
name: Deploy docs
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: 3.x
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```

## Folder Structure

```
HomeLab-Docs/
├── .github/workflows/deploy.yml
├── docs/
│   ├── assets/pdfs/          # session PDFs
│   ├── infra/                # hardware, proxmox
│   ├── network/              # vlans, opnsense, ddns, printer
│   ├── services/             # vaultwarden, homepage, etc.
│   ├── security/             # hardening docs
│   ├── vpn/                  # wireguard
│   ├── session-logs/         # session records
│   ├── meta/                 # this page
│   └── index.md
└── mkdocs.yml
```

## Issues & Fixes

| Issue | Fix |
|---|---|
| GitHub Actions 403 | Repo Settings → Actions → General → Read and write permissions |
| Files in `docs/docs/` | Moved with `mv` command |
| Grid cards not rendering | Added `attr_list`, `md_in_html` extensions |
| Icons not rendering | Added `pymdownx.emoji` extension |
| Password auth rejected | Switch to Personal Access Token |
