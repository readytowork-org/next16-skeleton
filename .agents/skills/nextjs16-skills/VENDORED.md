# Vendored skill

This skill's content (`SKILL.md`) is copied verbatim from a third-party,
community-authored source, not written for this repo. It covers Next.js
16-specific facts: breaking changes, the Turbopack default, async Request
APIs, and the upgrade path. It is a good match for this repo specifically,
since `package.json` pins `next: 16.0.7`.

- Source: https://github.com/gocallum/nextjs16-agent-skills
- Path: `skills/nextjs16-skills/SKILL.MD` (renamed to `SKILL.md` here for
  case-sensitive filesystem compatibility; the upstream file uses an
  uppercase `.MD` extension, inconsistent with every sibling skill in
  that repo)
- Vendored: 2026-09-18

License: unlike the `nestjs-best-practices` skill vendored into the
sibling `nest-backend-skeleton` repo (which self-declares MIT in its own
frontmatter), this skill's frontmatter and its source repository have no
license declaration at all as of vendoring (checked via the GitHub API).
Usage terms are therefore unclear. This is fine for internal use in a
private repo, but confirm licensing with the author before any broader
redistribution.

Do not hand-edit `SKILL.md` in place; if a correction is needed, either
fix it upstream and re-vendor, or note the deviation here instead of
silently diverging from the source.
