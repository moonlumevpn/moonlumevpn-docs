<p align="center">
  <h1 align="center">MoonlumeVPN Docs</h1>
  <p align="center"><b>Source-of-truth documentation for MoonlumeVPN</b></p>
  <p align="center">
    Content repo + Docusaurus engine repo + automatic rebuild trigger.
  </p>
  <p align="center">
    <img src="https://img.shields.io/badge/docs_content-markdown-blue" alt="Docs Content"/>
    <img src="https://img.shields.io/badge/site-docusaurus-2ea44f" alt="Docusaurus"/>
    <img src="https://img.shields.io/badge/default_locale-ru-orange" alt="Default Locale"/>
    <img src="https://img.shields.io/badge/i18n-ready-lightgrey" alt="i18n"/>
    <img src="https://img.shields.io/badge/registry-links.json-informational" alt="Registry"/>
    <img src="https://img.shields.io/badge/branch-production-black" alt="Production Branch"/>
  </p>
  <p align="center">
    <a href="#overview">Overview</a> •
    <a href="#architecture">Architecture</a> •
    <a href="#repository-structure">Structure</a> •
    <a href="#workflow">Workflow</a> •
    <a href="#setup">Setup</a> •
    <a href="#results">Result</a>
  </p>
</p>

---

## Overview
This repository (`moonlumevpn-docs`) stores documentation content in Markdown.

It works with a separate repository (`moonlumevpn-docs-site`) that:
- builds the static docs site with Docusaurus,
- publishes it to `https://docs.moonlumevpn.ru`,
- exposes app-readable links from `registry/links.json` as `/links.json`.

Russian (`ru`) is the default locale. Additional languages are supported via `i18n/`.

## Architecture
Two repositories are used:

1. `moonlumevpn-docs`
- content-only repository
- source of truth for docs
- includes machine-readable links registry

2. `moonlumevpn-docs-site`
- Docusaurus engine and deployment repository
- fetches this docs repo during CI
- builds and deploys the public site

## Repository Structure
```text
moonlumevpn-docs/
├─ docs/
│  └─ legal/
│     ├─ _category_.json
│     ├─ privacy_policy.md
│     ├─ public_offer.md
│     ├─ terms_of_use.md
│     └─ referral_program.md
├─ i18n/
│  └─ en/
│     └─ docusaurus-plugin-content-docs/
│        └─ current/
│           └─ legal/
│              └─ _category_.json
├─ registry/
│  └─ links.json
└─ README.md
```

`registry/links.json` is intended for bots/apps and currently contains legal document routes.

## Workflow
1. Update docs in `moonlumevpn-docs` (this repo).
2. Push to `production`.
3. `.github/workflows/trigger.yml` sends `repository_dispatch` (`docs-update`) to `moonlumevpn-docs-site`.
4. Engine repo rebuilds and deploys the site.

## Setup
### 1) Create Repositories
Create both repositories in your GitHub organization:
- `moonlumevpn-docs`
- `moonlumevpn-docs-site`

### 2) Configure Docs Content Repo
- Keep default language docs in `docs/`.
- Keep translated docs in `i18n/<locale>/docusaurus-plugin-content-docs/current/`.
- Maintain machine-readable links in `registry/links.json`.

### 3) Bootstrap Docusaurus Engine Repo
```bash
npx create-docusaurus@latest moonlumevpn-docs-site classic
cd moonlumevpn-docs-site
npm install
```

Set in `docusaurus.config.ts` (or `.js`):
```ts
url: 'https://docs.moonlumevpn.ru'
baseUrl: '/'
organizationName: 'moonlumevpn'
projectName: 'moonlumevpn-docs-site'

i18n: {
  defaultLocale: 'ru',
  locales: ['ru', 'en'],
}
```

### 4) Build Pipeline in Engine Repo
Create `.github/workflows/deploy.yml` in `moonlumevpn-docs-site`:

```yaml
name: Build & Deploy Docs

on:
  push:
    branches: [production]
  repository_dispatch:
    types: [docs-update]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout engine repo
        uses: actions/checkout@v4

      - name: Clone docs repo
        run: git clone https://github.com/moonlumevpn/moonlumevpn-docs.git external-docs

      - name: Replace docs folder
        run: |
          rm -rf docs
          mv external-docs/docs ./docs

      - name: Replace i18n folder if present
        run: |
          if [ -d external-docs/i18n ]; then
            rm -rf i18n
            mv external-docs/i18n ./i18n
          fi

      - name: Publish links registry for apps
        run: |
          mkdir -p static
          cp external-docs/registry/links.json static/links.json

      - name: Install dependencies
        run: npm ci

      - name: Build site
        run: npm run build

      - name: Deploy to GitHub Pages
        run: npm run deploy
        env:
          GIT_USER: github-actions
          USE_SSH: false
```

### 5) Configure Trigger Secret
In `moonlumevpn-docs`:
- `Settings` -> `Secrets and variables` -> `Actions`
- add `REPO_TOKEN` with `repo` scope

This repository uses that secret in `.github/workflows/trigger.yml` to notify the engine repo.

### 6) Configure GitHub Pages + DNS
In `moonlumevpn-docs-site`:
- `Settings` -> `Pages`
- source: `gh-pages` branch
- custom domain: `docs.moonlumevpn.ru`

In DNS:
- type: `CNAME`
- host: `docs`
- value: `moonlumevpn.github.io`

## Result
- Docs are edited in `moonlumevpn-docs`.
- Public docs are available at `https://docs.moonlumevpn.ru/`.
- App links are available at `https://docs.moonlumevpn.ru/links.json`.
- Russian is default locale, with i18n-ready structure for more languages.

## Good Practices
- Keep doc structure clean and predictable.
- Use PR-based review for content changes.
- Add optional checks like `markdownlint` and broken-link validation.
- Add preview deployments for PRs when needed.
