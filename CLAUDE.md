# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The official Next.js App Router course project (nextjs.org/learn). A dashboard app for managing invoices and customers, used as a learning exercise. It is not a production app.

## Commands

```bash
pnpm dev        # dev server with Turbopack at http://localhost:3000
pnpm build      # production build
pnpm start      # start production build
```

No lint or test scripts are configured.

## Environment setup

Copy `.env.example` to `.env.local` and fill in the values from a Vercel Postgres database:

```
POSTGRES_URL=
AUTH_SECRET=   # generate with: openssl rand -base64 32
AUTH_URL=http://localhost:3000/api/auth
```

Seed the database by hitting `GET /seed` once after setup.

## Architecture

**Stack:** Next.js 14 App Router · TypeScript · Tailwind CSS · `postgres` (raw SQL, no ORM) · NextAuth v5 (beta) · Zod

**Data layer** — `app/lib/data.ts` holds all database query functions. They use tagged template literals via the `postgres` package directly against Neon/Vercel Postgres over SSL. Amounts are stored as integer cents in the DB; `formatCurrency` in `app/lib/utils.ts` converts cents → USD string at display time.

**Types** — `app/lib/definitions.ts` has all hand-written TypeScript types. `LatestInvoiceRaw` vs `LatestInvoice` is the canonical example of the raw-DB vs formatted-display split used throughout.

**UI components** — `app/ui/` is organised by feature: `dashboard/`, `invoices/`, `customers/`. Shared primitives (`button.tsx`, `search.tsx`, `skeletons.tsx`) sit at the top level of `ui/`. Skeleton components in `skeletons.tsx` are used as `loading.tsx` fallbacks for Suspense boundaries.

**Route structure** — the app has a single root layout (`app/layout.tsx`) with no font or metadata set up yet. `app/seed/route.ts` is a one-shot GET handler that creates tables and inserts placeholder data. `app/query/route.ts` is a commented-out scratch route used during the course chapters.

**Search & pagination** — implemented via URL search params (`useSearchParams`, `useRouter`, `usePathname`). `use-debounce` is used in the search input to avoid hammering the DB on every keystroke.

**Auth** — NextAuth v5 beta (`next-auth@5.0.0-beta.25`). `AUTH_SECRET` and `AUTH_URL` must be set in `.env.local`.
