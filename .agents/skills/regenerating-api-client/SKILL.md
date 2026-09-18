---
name: regenerating-api-client
description: Regenerates this Next.js app's Axios API client from a backend's Swagger spec via swagger-typescript-api. Use when the backend API changed, or when code references a generated service under src/services/ that doesn't exist yet.
---

# Regenerating the API client

The generated client is not committed until someone runs this; if you hit
an import error for a service like `somethingApi` from `@/src/services`,
this is usually why (see `AGENTS.md`'s "Known inconsistencies" #3).

## Steps

1. Make sure `.env` has `NEXT_PUBLIC_APP_API_URL` pointing at a running,
   compatible backend that serves `/swagger/doc.json`.

2. Run:

   ```sh
   yarn swagger-gen
   ```

   This runs `src/services/codegen.mjs` (via `swagger-typescript-api`),
   which writes one file per Swagger tag into `src/services/`, plus a
   generated `src/services/index.ts` that imports `_httpClient` from
   `api-config.ts` and exports one camelCased const per tag (for example
   a tag generates something like `blogManagementApi`, matching how
   `src/utils/sitemap/*.ts` already reference these). Then it runs
   `yarn format`.

3. Review the diff under `src/services/`. Do not hand-edit generated
   files; re-run codegen instead if something needs to change on the
   client side (the fix usually belongs in the backend's Swagger spec).

4. If `src/services/api-config.ts` still references the unresolved
   `@skeleton/shared` import (see `AGENTS.md`'s "Known inconsistencies"
   #2), fix or replace the auth wiring there before relying on
   `_httpClient`'s request interceptor.

5. Run the verifying-changes-before-commit skill before treating the
   change as done.
