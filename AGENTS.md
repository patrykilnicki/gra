# Repository Guidelines

TribalWars is a small browser strategy game on the 10x Astro starter (Astro 7 SSR, React 19 islands, Tailwind 4, Supabase auth, Node standalone on Render). Scope is @context/foundation/prd.md; the stack hand-off is @context/foundation/tech-stack.md; deployment is @context/foundation/infrastructure.md.

## Hard rules

- Keep `output: "server"` in @astro.config.mjs. Do not set `prerender = true`. API handlers export uppercase `GET` or `POST`, as in @src/pages/api/auth/signin.ts.
- Do not add Next.js `"use client"` directives. Astro owns layout; React is only for interactive islands.
- Merge Tailwind classes with `cn()` from @src/lib/utils.ts. Do not concatenate class strings.
- New tables need RLS with a separate policy per operation and role. Name files `YYYYMMDDHHmmss_short_description.sql` under `supabase/migrations/`.
- Secrets are `SUPABASE_URL` and `SUPABASE_KEY`, server-only in @astro.config.mjs. Copy @.env.example to `.env`. Do not commit it. In production they are Render environment variables.
- Deploy target is a paid Render web service in Frankfurt (@render.yaml, adapter `node({ mode: "standalone" })`). Never use the Free plan (it sleeps), never set `NODE_VERSION` (`.nvmrc` must win), keep `HOST=0.0.0.0`. Game timers (battles, queues) live as Supabase rows, not in-process memory.
- Gate authenticated routes by adding the path to `PROTECTED_ROUTES` in @src/middleware.ts (currently `/dashboard` only).

## Build, test, and development

- `npm run dev` starts the `astro dev` server.
- `npm run lint` runs type-checked ESLint (@eslint.config.js). `npm run lint:fix` fixes.
- `npm run build` produces the SSR build. `npm start` runs it (`node ./dist/server/entry.mjs`, reads `HOST`, `PORT`, `SUPABASE_URL`, `SUPABASE_KEY` from the environment; locally use `node --env-file=.env ./dist/server/entry.mjs`). `astro preview` is not the production server.
- `npm run smoke` runs the auth smoke test (@scripts/smoke.mjs). `BASE_URL` defaults to `http://localhost:4321`.
- Use Node.js 22.14.0 (@.nvmrc). Local Supabase is `npx supabase start` (Docker).
- Deploys go out by push to `master` (Render autodeploy). Logs: `render logs -r SERVICE_ID --tail`. Rollback is a dashboard action and disables autodeploy; leave it, instance type, domains and key rotation to a human.

## Project structure

- `src/pages/` holds routes. Auth screens are `src/pages/auth/`. The protected example is @src/pages/dashboard.astro.
- `src/components/` mixes Astro and React. Auth islands are `src/components/auth/`. shadcn/ui ("new-york") is `src/components/ui/`; add pieces with `npx shadcn@latest add <name>` (@components.json).
- Helpers live in `src/lib/` (@src/lib/supabase.ts). Put new hooks in `src/hooks/` (`@/hooks` in @components.json) and shared types in `src/types.ts`.
- `@/*` maps to `./src/*` (@tsconfig.json).

## Coding style

Prettier (@.prettierrc.json): 2 spaces, double quotes, semicolons, print width 120. Run `npm run format` to rewrite the tree. ESLint errors on unused names unless they start with `_`; `no-console` is a warning. @.husky/pre-commit runs lint-staged from @package.json: `eslint --fix` on `*.{ts,tsx,astro}`, Prettier on `*.{json,css,md}`.

## Testing

There is no unit-test runner. The only behavior check is `npm run smoke`. CI (@.github/workflows/ci.yml) on push and pull request to `master` runs lint, `npx astro check`, and build (repository secrets for Supabase), then smoke against `npm start` with local Supabase.

## Commits and pull requests

This directory is not a git repository, so no commit-message convention exists yet. The required check is the `master` CI workflow above.
