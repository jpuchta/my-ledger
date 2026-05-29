# Repository Guidelines

my-ledger is a privacy-first household finance web app in pre-MVP scaffold: Astro 6 SSR on Cloudflare Workers, React 19 islands, Tailwind 4, Supabase auth. Product scope is in @context/foundation/prd.md; setup, Supabase, and deployment are in @DEVELOPMENT.md.

## Hard rules

- Do not commit `.env`, `.dev.vars`, or other secrets; copy from @.env.example.
- API routes under `src/pages/api/`: export uppercase `GET` / `POST` handlers (see @src/pages/api/auth/signin.ts). Add `export const prerender = false` on any route that must stay dynamic.
- New Supabase tables: add `supabase/migrations/YYYYMMDDHHmmss_short_description.sql` and enable RLS with per-operation, per-role policies.
- No Next.js directives (`"use client"`, etc.) on React islands.
- Merge Tailwind classes with `cn()` from @src/lib/utils.ts — never concatenate class strings manually.

## Project structure

- `src/pages/` — Astro pages; `src/pages/api/` — API endpoints.
- `src/components/` — Astro UI; `src/components/auth/` — interactive React forms; `src/components/ui/` — shadcn (new-york, @components.json).
- `src/lib/` — Supabase client (@src/lib/supabase.ts), helpers, future services.
- `src/middleware.ts` — session resolution and `PROTECTED_ROUTES` gate (extend the array for new private pages).
- `context/` — PRD and change notes (not runtime code).

Path alias `@/*` → `./src/*` per @tsconfig.json.

## Build, test, and development

Run on Node 22.14.0 (@.nvmrc):

- `npm run dev` — local dev server (Cloudflare workerd)
- `npm run build` — production SSR build
- `npm run lint` / `npm run lint:fix` — ESLint with type-checking (@eslint.config.js)
- `npm run format` — Prettier (@.prettierrc.json)

Husky + lint-staged: ESLint on `*.{ts,tsx,astro}`, Prettier on `*.{json,css,md}` before commit (@package.json).

No automated test runner is configured yet.

## Coding style

- Use `.astro` for static layout and pages; add React only where interactivity is required.
- Place shared hooks in `src/hooks/` (shadcn alias in @components.json).
- Place shared entity/DTO types in `src/types.ts` when introduced.
- Install UI primitives with `npx shadcn@latest add [name]` into `src/components/ui/`.

## Commits and pull requests

Recent commits use short imperative subjects (e.g. `Bootstrap project from 10x Astro Starter`). No enforced Conventional Commits prefix. PRs must pass CI on `main`: `npm run lint` and `npm run build` (@.github/workflows/ci.yml), with `SUPABASE_URL` and `SUPABASE_KEY` repository secrets for the build step.

## Configuration

Server-only secrets `SUPABASE_URL` and `SUPABASE_KEY` are declared in @astro.config.mjs `env.schema`. Local values go in `.env` (Node) and `.dev.vars` (Wrangler). Local Supabase: `npx supabase start` (Docker). Deploy: `npx wrangler deploy` (@wrangler.jsonc).
