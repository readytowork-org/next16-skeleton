---
name: adding-environment-variables
description: Adds a new environment variable to this Next.js app, keeping .example.env, local usage, and CircleCI deploy config in sync. Use when a change needs a new config value, secret, or feature flag read from the environment.
---

# Adding an environment variable

## Steps

1. Add it to `.example.env` with a placeholder value and a short comment.

2. Read it with `process.env.YOUR_VAR`. If the value must be readable in
   the browser (client components), prefix it `NEXT_PUBLIC_YOUR_VAR`, per
   Next.js convention; if it's server-only (API routes, `"use server"`
   actions), no prefix is needed and it stays out of the client bundle.

3. If local development needs a real value, add it to your own `.env`
   (not committed; see `.gitignore`).

4. If the deployed app needs it, add it to the `.env` file written by the
   `Initializing the Environment variables` step in
   `.circleci/config.yml`, in both `build_and_deploy_to_develop` and
   `build_and_deploy_to_production`, sourced from a CircleCI
   context/env var (see the `rtw_basic` / project context).

5. If it's required at runtime (like `CONSUMER_HOST` in
   `playwright.config.ts`, which throws if unset), make sure it's
   actually documented in `.example.env`; several existing required vars
   currently are not (see `AGENTS.md`'s "Known inconsistencies" #8).

6. Run the verifying-changes-before-commit skill before treating the
   change as done.
