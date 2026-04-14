# AGENTS

This repository supports human and AI-assisted documentation work.

## Purpose
- Keep edits focused on documentation content in `docs/`, `i18n/`, and `registry/`.
- Preserve Russian (`ru`) as the default locale unless a task explicitly targets translations.
- Keep changes small, reviewable, and safe for production deploys.

## Working Rules
- Prefer direct Markdown edits over structural churn.
- Keep routes and slugs stable unless the task explicitly requires route changes.
- Keep `registry/links.json` in sync with any docs structure change, including new, removed, renamed, or re-routed pages.
- When adding or changing legal docs, verify matching links in `registry/links.json`.
- Do not commit local cache artifacts.

## AI Context Layout
Use the tracked `ai/` directory for shareable project context:
- `ai/index.json`: pointers to key files and operating assumptions.
- `ai/context.md`: concise project memory for recurring tasks.
- `ai/README.md`: maintenance notes for this context set.

Use local `.ai/` for ephemeral cache and per-machine indexes:
- `.ai/` is intentionally git-ignored.
- Store temporary embeddings, retrieval indexes, and session notes there.

## Standard Flow
1. Read `README.md` and `ai/index.json`.
2. Apply the smallest valid change for the task.
3. Verify links/paths when touching navigation pages.
4. Update `ai/context.md` if the change alters long-term project conventions.
5. Commit with a clear, scoped message.
