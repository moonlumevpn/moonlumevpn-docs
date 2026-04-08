MoonlumeVPN Documentation Platform

This repository stores the source-of-truth documentation for MoonlumeVPN. It is designed to work with a separate Docusaurus engine repo that builds and deploys a static site.

Multilanguage is supported. Russian is the default language.

Architecture Overview
We use two repositories:

1) Docs Content Repo
Name: `moonlumevpn-docs`
Purpose: Store all documentation in Markdown
Audience: Developers and contributors
Source of truth for docs content

2) Docs Engine Repo
Name: `moonlumevpn-docs-site`
Built with Docusaurus
Pulls content from `moonlumevpn-docs` during CI build
Builds and deploys the static site
Public URL: `https://docs.moonlumevpn.ru`

Workflow
1. Update docs in `moonlumevpn-docs`
2. GitHub Action in this repo triggers a rebuild in `moonlumevpn-docs-site`
3. The site is rebuilt and deployed automatically

Repository Structure
Russian is the default locale and lives in `docs/`.
Additional locales live under `i18n/<locale>/docusaurus-plugin-content-docs/current/`.

moonlumevpn-docs/
  docs/
    privacy_policy.md
    public_offer.md
    terms_of_use.md
  i18n/
    en/
      docusaurus-plugin-content-docs/
        current/
          privacy_policy.md
          public_offer.md
          terms_of_use.md
  README.md

Setup Instructions

Step 1 - Create Repositories
Create two repos in your GitHub org `moonlumevpn`:
`moonlumevpn-docs`
`moonlumevpn-docs-site`

Step 2 - Setup Docs Repository
Keep default language docs inside `docs/`. Place translated docs under `i18n/<locale>/docusaurus-plugin-content-docs/current/`.

Step 3 - Setup Docusaurus Site
Initialize Docusaurus:
`npx create-docusaurus@latest moonlumevpn-docs-site classic`
`cd moonlumevpn-docs-site`
`npm install`

Update `docusaurus.config.ts` (or `.js`):
`url: 'https://docs.moonlumevpn.ru'`
`baseUrl: '/'`
`organizationName: 'moonlumevpn'`
`projectName: 'moonlumevpn-docs-site'`

Enable i18n (Russian default):
`i18n: {`
`  defaultLocale: 'ru',`
`  locales: ['ru', 'en'],`
`}`

Step 4 - Add Docs Fetch in Build
In the engine repo, configure GitHub Actions to pull this repo during build.
Create `.github/workflows/deploy.yml`:

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
        run: |
          git clone https://github.com/moonlumevpn/moonlumevpn-docs.git external-docs

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

      - name: Install dependencies
        run: npm ci

      - name: Build site
        run: npm run build

      - name: Deploy to GitHub Pages
        run: npm run deploy
        env:
          GIT_USER: github-actions
          USE_SSH: false

Step 5 - Trigger Rebuild from Docs Repo
This repo contains the GitHub Action at `.github/workflows/trigger.yml` that sends a `repository_dispatch` event to the engine repo on every push to `production`.

Step 6 - Add GitHub Token
In `moonlumevpn-docs` repo:
Settings -> Secrets -> Actions
Add new secret: `REPO_TOKEN`
Token permissions: `repo` (required to trigger workflows in a different repo)

Step 7 - Configure GitHub Pages
In `moonlumevpn-docs-site`:
Settings -> Pages
Source: `gh-pages` branch
Custom domain: `docs.moonlumevpn.ru`

Step 8 - Configure DNS
In your DNS provider:
Type: `CNAME`
Name: `docs`
Value: `moonlumevpn.github.io`

Final Result
Docs live in `moonlumevpn-docs`
Site auto-updates on every push
Hosted at `https://docs.moonlumevpn.ru/`
Multilanguage docs work with Russian as the default locale

Notes and Best Practices
Keep docs clean and structured
Use PRs for documentation changes
Add linting (optional: `markdownlint`)
Version docs later using Docusaurus versioning

Optional Improvements
Add preview builds for PRs
Add search (Algolia or local search)
Add CI checks for broken links
