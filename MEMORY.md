# Memory

A log of durable progress, decisions, and next steps for this repo, meant
for both humans and AI agents picking up work here. Read it before
starting a session.

Update it only for milestones: a notable decision, a completed body of
work, or an open item worth remembering, not a line for every edit in a
session. If in doubt, don't add an entry; `git log` already covers routine
changes.

## Next steps / open items

- `README.md` is stale `create-next-app` boilerplate (mentions MUI, which
  isn't used); not yet fixed.
- The ten "Known inconsistencies" in `AGENTS.md` (broken `BackButton`
  import, dead `api-config.ts`, sitemap utils depending on ungenerated
  API client code, incomplete root component barrel, a likely-dead
  `.eslintrc.js`, a translation-fetch route that writes files the app
  doesn't read, an unmodified Playwright starter test, an undocumented
  required `CONSUMER_HOST` env var, a broken `yarn lint` script, and a
  deprecated `middleware.ts`) are documented but not yet cleaned up. In
  particular, #2 and #3 mean a full `yarn build` currently fails in this
  checkout independent of unrelated changes.
- Despite being named `next16-skeleton`, this checkout contains a
  specific product's code (`minano-user` / GTM IDs / job-site sitemap
  logic), not a clean generic starter; worth stripping down if this repo
  is meant to be reused as a template.

## Decisions

- **2026-09-18**: Skills are authored once under `.agents/skills/` and
  exposed to individual agents via symlinks (`.claude/skills`,
  `.codex/skills`), instead of duplicating SKILL.md files per tool,
  matching the convention already used in `go-gin-skeleton` and
  `nest-backend-skeleton`. Caveat: needs `core.symlinks` support on
  checkout, so the symlinks can check out broken on Windows without it.
- **2026-09-18**: This repo has no Makefile, so the symlink-refresh
  command is a `yarn link-skills` script (`scripts/link-agent-skills.sh`)
  instead of a `make` target, matching this repo's actual tooling rather
  than importing a convention from the other two repos.
- **2026-09-18**: This repo intentionally has no `CLAUDE.md`, so Claude
  Code does not auto-load `AGENTS.md` at session start here (an agent
  needs to read it itself).
- **2026-09-18**: New documentation in this repo avoids em dashes;
  existing docs are not reformatted to remove them.
- **2026-09-18**: Vendored a third-party community skill, `nextjs16-skills`
  (github.com/gocallum/nextjs16-agent-skills), chosen specifically because
  it's version-matched to this repo's `next: 16.0.7`, unlike a more
  generic App Router best-practices skill. Cross-referencing it against
  Next.js's own v16 upgrade guide surfaced two more real issues, now in
  "Known inconsistencies" #9-10: `yarn lint` is broken (`next lint` was
  removed in v16), and `src/app/middleware.ts` uses a deprecated filename
  (`proxy.ts` is the v16 convention). Unlike the `nestjs-best-practices`
  skill vendored into the sibling `nest-backend-skeleton` repo, this one
  has no declared license anywhere; see its `VENDORED.md`.

## Progress log

- **2026-09-18**: Cloned from `readytowork-org/next16-skeleton` (sibling
  of `go-gin-skeleton` and `nest-backend-skeleton`, same conventions
  applied for consistency). Added `AGENTS.md`, written from reading the
  actual source rather than `README.md`, which turned out to be stale
  `create-next-app` boilerplate; found and documented eight real broken
  or dangling pieces of code in the process (see "Next steps"). Added
  four repo-specific Claude Code skills under `.agents/skills/`
  (`scaffolding-atomic-components`, `regenerating-api-client`,
  `adding-environment-variables`, `verifying-changes-before-commit`) and
  a `yarn link-skills` command, exposed via `.claude/skills` and
  `.codex/skills` symlinks. No application code or existing documentation
  was modified; nothing has been pushed.
