---
name: scaffolding-atomic-components
description: Scaffolds a new atom, molecule, organism, or template component in this Next.js app's atomic-design structure and wires up its barrel exports. Use when adding a new UI component under src/components/.
---

# Scaffolding an atomic-design component

Full context on this repo's real structure and known quirks lives in
`AGENTS.md` at the repo root. Read it first, especially "Known
inconsistencies" #4 (the root `src/components/index.ts` barrel currently
skips `atoms` and `molecules`) before assuming a root-barrel import works.

## Steps

1. Pick the right layer:
   - `atoms/`: smallest reusable piece (a button, an icon wrapper, a
     switch). See `src/components/atoms/BackButton/` for the pattern.
   - `molecules/`: a small composition of atoms. Currently empty in this
     repo; you are likely creating the first real one.
   - `organisms/`: a larger composed piece, often with its own state or
     provider (see `QueryClientProviders`).
   - `template/`: a full page-level view, split into `index.ts` (barrel)
     and `view.tsx` (the component), see `NotFoundPage`.

2. Create `src/components/<layer>/<ComponentName>/index.tsx` (or
   `index.ts` + `view.tsx` for a `template`). Add `"use client"` at the
   top only if the component needs interactivity, state, or browser APIs;
   leave it off for server components.

3. Style with Tailwind utility classes inline, matching the rest of the
   codebase; there are no CSS modules here.

4. If the component needs copy, use `useTranslations()` from `next-intl`
   and add the keys to **both** `src/messages/en/common.json` and
   `src/messages/ja/common.json`.

5. Export it from `src/components/<layer>/index.ts` (or `index.tsx`).
   Only add it to the root `src/components/index.ts` barrel if that
   layer is already re-exported there (currently only `organisms` and
   `template` are); otherwise import from the specific layer path
   (`@/src/components/atoms`, etc.).

6. Run the verifying-changes-before-commit skill before treating the
   component as done.
