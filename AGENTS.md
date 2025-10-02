# Repository Guidelines

## Project Structure & Module Organization
- `app/` hosts the Next.js App Router; `page.tsx` is the default landing view and `app/api/auth/[...nextauth]/route.ts` exposes the Auth.js handler.
- `lib/` contains database wiring; use `lib/db.ts` for cached D1 clients and adjust tables in `lib/schema/d1.ts` when the Auth.js model changes.
- Root modules (`auth.ts`, `middleware.ts`) configure session handling across edge routes.
- Deployment and database configs live beside the root (`wrangler.jsonc`, `drizzle-*.config.ts`, `open-next.config.ts`); static assets belong in `public/`.

## Build, Test, and Development Commands
- `pnpm install` syncs dependencies; avoid mixing package managers.
- `pnpm dev` starts Turbopack locally; `pnpm build` followed by `pnpm start` mirrors production.
- `pnpm lint` runs `next lint`; fix warnings before committing.
- Schema work: `pnpm exec drizzle-kit generate --config drizzle-dev.config.ts` then `pnpm exec drizzle-kit migrate --config drizzle-dev.config.ts` to update the local D1 emulator.
- `pnpm cf-typegen` refreshes typed Cloudflare bindings; include the updated `cloudflare-env.d.ts`. Use `pnpm deploy` or `pnpm preview` only after a clean build.

## Coding Style & Naming Conventions
- TypeScript + ES modules with 2-space indentation and trailing commas matching existing files; run the built-in formatter before pushing.
- Name routes by feature (`app/(auth)/signin/page.tsx`) and colocate utilities under `lib/feature/*`.
- Align Drizzle column names with Auth.js expectations; prefer camelCase except when targeting existing snake_case tables.

## Testing Guidelines
- No harness is bundled yet; introduce `vitest` or `jest` as needed and place specs next to source (`*.test.ts`).
- Before merging, run `pnpm lint`, a smoke `pnpm build`, and verify migrations via `drizzle-kit migrate`.

## Commit & Pull Request Guidelines
- Follow Conventional Commit prefixes in history (`feat`, `chore`, `fix`, `docs`), using scopes for clarity (`feat(auth): add passkeys`).
- Each PR should summarize behavior changes, call out schema or env updates, link the tracking issue, and include screenshots or curl output for UI/API edits.
- Keep branches rebased on `main`; document skipped checks or manual steps in the PR body.

## Cloudflare & D1 Configuration
- Store secrets in `.dev.vars`; mirror temporary values in `.env.local` only when generating `AUTH_SECRET`, and purge them before committing.
- Create or bind the D1 database with `pnpm dlx wrangler d1 create` using the `DB` binding expected by `lib/db.ts`.
- Run remote migrations with `drizzle-prod.config.ts` and ensure `wrangler.jsonc` uses the current `database_id` before deploying.
