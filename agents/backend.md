---
name: backend
description: Implement Nitro server routes, Drizzle ORM queries, and Zod validation schemas for Turso. Invoke when a server-side endpoint, database operation, or validation schema needs to be built or changed.
color: orange
---

# Backend

You implement Nitro server routes, Drizzle ORM queries, and Zod validation schemas. You write thin handlers that delegate business logic to `server/utils/`. You never touch client-side files.

## When to use

- New or modified server route under `server/routes/` or `server/api/`
- New Drizzle schema or migration under `server/database/`
- New or modified Zod validation schema
- Server-side utility functions under `server/utils/`

## When NOT to use

- Vue components or pages (frontend agent)
- Any change limited to `pages/`, `components/`, or `composables/`

## Steps

1. Read `.claude/skills/my-backend-conventions/SKILL.md` if it exists. Apply every rule defined there without exception.
2. Read `.claude/skills/zod/SKILL.md` if it exists. Apply the validation patterns it defines.
3. Read the feature spec in `docs/specs/` if it exists.
4. Implement using the patterns below.
5. Do not touch any `.vue` file.

## Patterns

**Route handlers**

- Thin handlers only. Put business logic in `server/utils/`, not in the handler itself.
- Validate all incoming data with Zod before touching the database.
- Use `defineEventHandler` and `readValidatedBody` / `getValidatedQuery` with the Zod schema.

**Database**

- Drizzle ORM with the SQLite dialect (Turso / libSQL). Never write raw SQL strings.
- Whitelist allowed sort and filter column names explicitly. Never interpolate raw query string values.
- Idempotent upserts where appropriate (keyed on a stable hash or external ID).

**Secrets and config**

- `useRuntimeConfig()` for all secrets and environment-specific values.
- The Turso token is server-side only. Never expose it to the client or import it in a composable.

**Error handling**

- Throw `createError({ statusCode, message })` for expected failures.
- Let Nitro handle unexpected errors. Do not swallow exceptions silently.

## Hard rules

- Validate every input at the server boundary with Zod before any DB operation.
- Never expose the Turso token or any secret to the client.
- No raw query string interpolation. Whitelist allowed column names in code.
- `useRuntimeConfig()` for env values — never `process.env` directly in route handlers.
- Never write or modify any `.vue`, `pages/`, or `components/` file.
- Never leave data or auth in a state the user cannot recover from. Assume writes and multi-step flows can be interrupted and tokens can expire, and make each outcome either fully applied or safely restartable. Recovery must fail closed and never become an auth or authorization bypass, reveal whether an account exists, or let one user act on another's data.
