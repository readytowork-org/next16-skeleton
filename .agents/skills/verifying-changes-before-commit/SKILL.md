---
name: verifying-changes-before-commit
description: Runs this Next.js app's lint, format, build, and test checks before a change is considered done. Use before finishing any code change, opening a PR, or when asked to verify or double-check a change.
---

# Verifying changes before commit

Copy this checklist and check items off as you go:

```
- [ ] 1. npx eslint .
- [ ] 2. yarn format
- [ ] 3. yarn build
- [ ] 4. npx playwright test, if you touched UI behavior covered by a spec
- [ ] 5. yarn swagger-gen, if the backend API surface changed
- [ ] 6. Docs updated, if user-facing behavior changed
```

**Step 1-2: Lint and format**

```sh
npx eslint .
yarn format
```

Not `yarn lint`: it runs `next lint`, which Next.js 16 removed (see
`AGENTS.md`'s "Known inconsistencies" #9). `npx eslint .` reads the same
`eslint.config.mjs`.

**Step 3: Build**

```sh
yarn build
```

This is also the closest thing to a type-check (no separate `tsc`
script). Note: as of this writing, a full build in this checkout already
fails independent of most changes, because of `AGENTS.md`'s "Known
inconsistencies" #2 (`api-config.ts`'s missing `@skeleton/shared` import)
and #3 (`src/utils/sitemap/*.ts` depending on an ungenerated API client).
A build failure pointing at one of those specific files is a pre-existing
issue, not necessarily something your change introduced; a build failure
anywhere else is worth investigating as your own.

**Step 4: Playwright**

There is no `yarn test`/`test:e2e` script; run `npx playwright test`
directly. Requires `CONSUMER_HOST` in `.env` (see
`playwright.config.ts`). The only existing spec,
`tests/example.spec.ts`, is the unmodified Playwright starter and tests
`playwright.dev`, not this app; don't treat it as a working example.

**Step 5: API client**

If the change depends on a backend API change, use the
regenerating-api-client skill instead of hand-editing anything under
`src/services/`.

**Step 6: Documentation**

If the change alters documented, user-facing behavior, update
`AGENTS.md`. `README.md` is known to be stale boilerplate (see
`AGENTS.md`'s "Documentation map"); fixing it opportunistically is
welcome but not required for every change.
