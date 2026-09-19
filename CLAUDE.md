# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Project

Arcade Vault — a platform for playing games online and competing for high scores (per README.md, in Spanish). The repository is currently just the `create-next-app` scaffold; no app-specific routes, components, or data layer exist yet beyond the default template in `app/`.

The README documents an intended **Spec Driven Design** workflow (`/spec` and `/spec-impl`) from the [Klerith/fernando-skills](https://github.com/Klerith/fernando-skills) skill pack, installed via `npx skills@latest add Klerith/fernando-skills`. That skill pack is not currently installed in this repo (no `.claude/skills` present) — if asked to follow spec-driven workflows, check whether it has been installed since, or offer to install it first.

## Commands

```bash
npm run dev     # start the dev server (Turbopack, on by default in Next 16)
npm run build   # production build (Turbopack, on by default in Next 16)
npm run start   # run the production build
npm run lint    # eslint (flat config); there is no `next lint` in Next 16 — it was removed
```

There is no test setup in this repo yet (no test runner in `package.json`).

## Architecture

Standard Next.js **App Router** layout:

- `app/layout.tsx` — root layout, loads Geist fonts, exports `metadata`.
- `app/page.tsx` — home page (still the default `create-next-app` starter content).
- `app/globals.css` — Tailwind v4 (`@import "tailwindcss"`) with theme tokens defined via `@theme inline` and CSS custom properties for light/dark mode.
- `public/` — static assets (default starter SVGs).
- Path alias `@/*` → repo root (`tsconfig.json`).

## Working with this Next.js version — read before writing code

This repo pins **Next.js 16.3.5**, which has real breaking changes vs. what's in most training data. `AGENTS.md` (imported above) is the canonical instruction: **read the matching guide under `node_modules/next/dist/docs/` before writing App Router code**, especially before touching routing, caching, images, or config. That directory mirrors nextjs.org/docs and is version-matched to the installed `next` package — trust it over prior knowledge.

Gotchas already visible in this codebase or otherwise easy to trip over:

- **`LayoutProps`/`PageProps`/`RouteContext` global type helpers**: `app/layout.tsx` already uses `LayoutProps<"/">` instead of a hand-written `{ children: React.ReactNode }` prop type. These are ambient, auto-generated types (no import needed) — use `PageProps<'/route'>` and `LayoutProps<'/route'>` for new pages/layouts instead of writing prop types by hand. Regenerate with `npx next typegen` if types seem stale.
- **Async Request APIs are fully async, no sync fallback**: `cookies()`, `headers()`, `draftMode()`, and `params`/`searchParams` must always be `await`ed — the Next 15 synchronous compatibility shim is gone in 16.
- **`middleware.ts` → `proxy.ts`**: the file convention and exported function are renamed (`proxy`, not `middleware`); the `edge` runtime is not supported in `proxy` (only `nodejs`).
- **Turbopack is the default** for both `next dev` and `next build` (no `--turbopack` flag needed); a custom Webpack config will fail `next build` unless you pass `--webpack` or migrate the config.
- **`revalidateTag(tag)` now requires a second `cacheLife` profile argument** (e.g. `revalidateTag('posts', 'max')`); use `updateTag` from `next/cache` instead for read-your-writes semantics in Server Actions.
- **Parallel route slots require an explicit `default.js`** — builds fail without one.
- **`next/image` defaults changed**: `minimumCacheTTL` is now 4h (was 60s), `qualities` defaults to `[75]` only, local images with query strings need `images.localPatterns[].search` configured, and `images.domains` is deprecated in favor of `images.remotePatterns`.
- **ESLint is flat-config only** (`eslint.config.mjs`, already set up in this repo) — there is no `next lint` command or `eslint` key in `next.config.ts` anymore.
