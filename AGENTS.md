# Agent Guide

This file follows [AGENTS.md](https://agents.md), an open, tool-agnostic
format for giving AI coding agents project context. It is read by agents
including OpenAI Codex, Cursor, Windsurf, Aider, GitHub Copilot, Google
Gemini CLI / Jules, Devin, Zed, and others. This repo does not ship a
`CLAUDE.md`, so a Claude Code session does not auto-load this file at
startup; read it yourself at the start of a session, or ask the user to
point you at it.

It captures the conventions that are not obvious from a single file, plus
several real, verified inconsistencies already in this codebase that are
easy to copy by accident. Read this before making changes. If something
here turns out to be wrong or stale, fix it as part of your change.

## What this project is

A Next.js 16 (App Router) frontend: React 19, TypeScript, Tailwind CSS,
`next-intl` for i18n (English/Japanese), TanStack Query, Axios via a
generated Swagger API client, Playwright for e2e, deployed to Google App
Engine. Package manager: Yarn.

`package.json`'s `name` is `minano-user` and `app.yaml`'s `service` is
`consumer`; despite the repo being named `next16-skeleton`, this checkout
currently contains a specific product's code (hardcoded Google
Tag Manager / Ads IDs in `GlobalTagManager`, job/blog sitemap logic,
Japanese-market copy), not a clean generic starter. Strip that out if
reusing this as a template for an unrelated project.

## Documentation map

`README.md` is the default `create-next-app` boilerplate and is
**out of date**: it says the project uses MUI components and theme config
from `readytowork-org/mui-theme-config`, but there is no MUI dependency in
`package.json` at all; the app actually uses Tailwind CSS. Treat this file
(`AGENTS.md`) as authoritative for actual structure and commands.

| File                        | Covers                                                                |
|------------------------------|--------------------------------------------------------------------|
| `README.md`                 | Default `create-next-app` boilerplate (stale, see above).           |
| `AGENTS.md` (this file)     | Agent-facing guide: real architecture, conventions, commands.       |
| `MEMORY.md`                 | Running log of progress, decisions, and next steps. Update it as you work; read it first to pick up where a previous session left off. |
| `.agents/skills/*/SKILL.md` | Canonical skill definitions: four repo-specific workflow skills, plus a vendored `nextjs16-skills` reference (see its `VENDORED.md`). `.claude/skills` and `.codex/skills` are symlinks to this directory; edit skills only under `.agents/skills/`. |

If you add a new documentation file, list it here too.

## Repository layout

| Path                        | Purpose                                                                 |
|------------------------------|--------------------------------------------------------------------------|
| `src/app/`                  | Next.js App Router: `layout.tsx`, `page.tsx`, `not-found.tsx`, `middleware.ts` (www/trailing-slash redirects; deprecated filename in Next 16, see "Known inconsistencies"), `api/fetch-translations/` (see "Known inconsistencies"). |
| `src/components/`           | Atomic design: `atoms/`, `molecules/`, `organisms/`, `template/`, each with a barrel `index.ts`/`index.tsx`. The root `src/components/index.ts` only re-exports `organisms` and `template`, not `atoms` or `molecules` (see "Known inconsistencies"). |
| `src/services/`             | `http-client.ts` (generated base Axios client), `api-config.ts` (unused, see "Known inconsistencies"), `locale.ts` (server actions for the locale cookie), `codegen.mjs` (drives `yarn swagger-gen`, writes generated per-tag API clients and `index.ts` here; not committed until run). |
| `src/i18n/`                 | `next-intl` config: `config.ts` (locales `en`/`ja`, default `ja`), `request.ts` (loads `src/messages/<locale>/common.json`). |
| `src/messages/`             | Translation JSON, one `common.json` per locale under `en/` and `ja/`. |
| `src/utils/`                | Hooks (`hook/`), `sitemap/` (depends on the generated API client, see "Known inconsistencies"), misc helpers. |
| `src/types/`                | Shared TypeScript types. |
| `src/fonts/`                | `next/font/local` setup. |
| `public/`                  | Static assets. |
| `tests/`                    | Playwright tests; `example.spec.ts` is the unmodified Playwright starter test (see "Known inconsistencies"). |
| `.circleci/config.yml`     | CI: build-only on non-develop branches, build+deploy to GAE on `develop` and version tags. |

## Path aliases

`tsconfig.json` defines only `@/*` -> `./*` (repo root, so `@/src/...`
reaches `src/`). Most files use `@/src/...` consistently. One file,
`BackButton`, imports `@/icons/back-icon.svg`, but there is no `icons/`
directory anywhere in the repo; that import does not resolve (see "Known
inconsistencies").

## The component pattern

Follow the existing atomic-design structure:

- `atoms/<Name>/index.tsx`: smallest reusable pieces (see `BackToTop`,
  `LanguageSwitch`, `GlobalTagManager`).
- `molecules/`: currently empty (`src/components/molecules/index.ts` has
  no exports); some `src/utils/sitemap/` files import a `FindYourJob`
  molecule that does not exist yet (see "Known inconsistencies").
- `organisms/<Name>/index.tsx`: composed pieces, e.g. `QueryClientProviders`.
- `template/<Name>/`: page-level views, split into `index.ts` (barrel) and
  `view.tsx` (the actual component), e.g. `NotFoundPage`.

Client components start with `"use client"`. Styling is Tailwind utility
classes inline; there are no CSS modules in this codebase. Translations go
through `useTranslations()` from `next-intl`, keyed into
`src/messages/<locale>/common.json`.

## Known inconsistencies

Read this before touching routing, the component barrels, or the
generated API client; all of these are real, verified quirks in the
current code, not hypotheticals.

1. **`BackButton` has a broken import.** `src/components/atoms/BackButton/index.tsx`
   imports `BackIcon from "@/icons/back-icon.svg"`, but no `icons/`
   directory exists anywhere in the repo (confirmed by search). This
   import cannot resolve as written.
2. **`api-config.ts` is dead code with a missing dependency.**
   `src/services/api-config.ts` imports `auth` from `@skeleton/shared`, a
   package that does not exist in `package.json`, `node_modules`, or
   anywhere else in this repo, and calls Firebase's `getIdToken()` even
   though `firebase` is not a dependency either. Nothing in `src/`
   imports `api-config.ts` (confirmed by search); it is unused. Do not
   build on it as-is; either wire up real auth or delete it.
3. **`src/utils/sitemap/*.ts` depend on ungenerated API client code.**
   `blogAndBasicPages.ts` and `jobs.ts` import `publicBlogManagementApi`
   / `publicJobManagementApi` from `@/src/services`, and
   `jobSearchConditions.ts` imports from `@/src/components/molecules/FindYourJob/types`,
   which does not exist. These only resolve after running `yarn swagger-gen`
   against a real, compatible backend (see "Commands") that produces
   matching service names, and after a `FindYourJob` molecule is built.
   In this checkout, neither exists, so these files do not currently
   compile.
4. **Root component barrel skips `atoms` and `molecules`.**
   `src/components/index.ts` only re-exports `organisms` and `template`.
   Code importing from `@/src/components` (for example the sitemap utils
   above) that expects atoms/molecules exports from there will not find
   them; import from the specific subdirectory instead
   (`@/src/components/atoms`, etc.) until/unless the root barrel is fixed.
5. **Two ESLint configs, one is dead.** Both `.eslintrc.js` (legacy
   format) and `eslint.config.mjs` (flat config) exist with overlapping
   rules. ESLint 9 (used here) discovers flat config by default, so
   `.eslintrc.js` is very likely unused. Edit `eslint.config.mjs`; treat
   `.eslintrc.js` as a stale leftover unless you confirm otherwise.
6. **`fetch-translations` writes files the app doesn't read.**
   `src/app/api/fetch-translations/route.ts` pulls a Google Sheet and
   writes flat `src/messages/en.json` / `src/messages/ja.json`, but
   `src/i18n/request.ts` actually loads `src/messages/<locale>/common.json`
   (nested per-locale directories). Running this route does not update
   the translations the app serves.
7. **`tests/example.spec.ts` is the unmodified Playwright starter**,
   testing `playwright.dev` itself, not this app. There is also no
   `test`/`e2e` script in `package.json` and no test step in
   `.circleci/config.yml`; running Playwright means invoking it directly
   (see "Commands").
8. **`playwright.config.ts` requires `CONSUMER_HOST`** (throws if unset),
   which is not documented in `.example.env`.
9. **`yarn lint` is broken on this Next.js version.** `package.json`
   defines `"lint": "next lint"`, but Next.js 16 removed the `next lint`
   command entirely (confirmed in the official v16 upgrade guide); use
   ESLint directly instead, for example `npx eslint .`, until the script
   is updated. `next build` also no longer runs linting as a side effect.
10. **`src/app/middleware.ts` uses a deprecated Next.js 16 convention.**
    Next.js 16 deprecates `middleware.ts`/`export function middleware` in
    favor of `proxy.ts`/`export function proxy` (same behavior, clearer
    naming; `proxy` only supports the `nodejs` runtime, not `edge`). It
    still works but should be renamed when convenient; see the
    `nextjs16-skills` skill for the exact migration.

## Adding an environment variable

Claude Code users: the `adding-environment-variables` skill runs this as
a guided checklist.

1. Add it to `.example.env` with a placeholder value and a short comment.
2. Read it via `process.env.YOUR_VAR` (client-exposed vars must be
   prefixed `NEXT_PUBLIC_`, per Next.js convention; see the existing
   `NEXT_PUBLIC_APP_API_URL`, `NEXT_PUBLIC_SPREADSHEET_ID`).
3. If it is needed at deploy time, add it to the `.env` file written by
   the `Initializing the Environment variables` step in
   `.circleci/config.yml` (`build_and_deploy_to_develop` and
   `build_and_deploy_to_production` jobs), sourced from a CircleCI
   context/env var.
4. If local development needs a real value, add it to your own `.env`
   (not committed; see `.gitignore`).

## Regenerating the API client

Claude Code users: the `regenerating-api-client` skill runs this as a
guided checklist.

`yarn swagger-gen` (`node --env-file .env src/services/codegen.mjs && yarn run format`)
fetches `${NEXT_PUBLIC_APP_API_URL}/swagger/doc.json` and regenerates
per-tag API client files plus `src/services/index.ts` into
`src/services/`, exporting one camelCased const per tag (for example a
`BlogManagement` tag becomes `blogManagementApi` style; see
"Known inconsistencies" #3 for names already referenced in this repo).
`NEXT_PUBLIC_APP_API_URL` in `.env` must point at a running, compatible
backend for this to work.

## Commands

Reflects `package.json` as it actually is today; do not trust
`README.md` (see "Documentation map").

| Command                | What it does                                                             |
|--------------------------|-----------------------------------------------------------------------|
| `yarn dev`              | Dev server with Turbopack (`next dev --turbo`).                        |
| `yarn build`            | Production build (`next build`).                                       |
| `yarn start`            | Runs the production build (`next start`).                              |
| `yarn lint`             | Runs `next lint`, which is removed in Next.js 16 and will fail (see "Known inconsistencies" #9). Use `npx eslint .` (reads `eslint.config.mjs`) until the script is fixed. |
| `yarn format`           | `prettier --check ./ --write`, then `eslint --fix`.                     |
| `yarn swagger-gen`      | Regenerates the API client (see "Regenerating the API client").        |
| `yarn link-skills`      | (Re)creates the `.claude/skills` and `.codex/skills` symlinks pointing at `.agents/skills`. Safe to re-run any time, e.g. after a fresh clone or if a symlink goes missing. Pass extra agent names to link more, e.g. `yarn link-skills cursor .windsurf`. |
| `npx playwright test`  | Runs the Playwright suite directly; there is no `yarn test`/`test:e2e` script (see "Known inconsistencies"). Requires `CONSUMER_HOST` in `.env`. |

## Testing

Claude Code users: the `verifying-changes-before-commit` skill runs the
checklist below before a change is considered done.

There is no meaningful existing test coverage to model new tests on: the
only spec file is the unmodified Playwright starter (see "Known
inconsistencies" #7). When adding tests, put Playwright specs under
`tests/`, and run them with `npx playwright test` directly since no
package.json script wraps it yet.

## Before you finish a change

1. `npx eslint .` (not `yarn lint`, see "Known inconsistencies" #9) and
   `yarn format`.
2. `yarn build` succeeds (this is also the closest thing to a type-check;
   note "Known inconsistencies" #2 and #3 already fail a full build in
   this checkout independent of your change, until fixed).
3. If you touched Playwright specs, run `npx playwright test`.
4. If you changed the API surface consumed by the frontend, re-run
   `yarn swagger-gen` (see above) rather than hand-editing generated files
   under `src/services/`.
5. If you changed documented, user-facing behavior, consider fixing
   `README.md` while you're there, since it's known stale (see
   "Documentation map"). Do not use em dashes in new documentation
   content.
