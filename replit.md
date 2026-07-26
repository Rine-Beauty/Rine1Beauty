# Rine Beauty

An Arabic (RTL) skincare and haircare boutique catalog — customers browse products and check out by opening a prefilled WhatsApp conversation with the store owner; the owner manages branding, categories, and products from a private admin dashboard.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server
- `pnpm --filter @workspace/rine-beauty run dev` — run the storefront/admin web app
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string; object storage env vars (`DEFAULT_OBJECT_STORAGE_BUCKET_ID`, `PRIVATE_OBJECT_DIR`, `PUBLIC_OBJECT_SEARCH_PATHS`) are already provisioned.

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite (artifact `artifacts/rine-beauty`), wouter router, TanStack Query
- API: Express 5 (artifact `artifacts/api-server`)
- DB: PostgreSQL + Drizzle ORM
- Auth: Replit Auth (OIDC), single-owner admin allowlist by email
- Storage: Replit Object Storage (product/logo/banner images)
- Validation: Zod (`zod/v4` in schema authoring; generated client is zod v3-compatible), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)

## Where things live

- API contract: `lib/api-spec/openapi.yaml` (source of truth) → codegen → `lib/api-client-react/src/generated/` (React Query hooks) and `lib/api-zod/src/generated/` (Zod schemas used by the API server)
- DB schema: `lib/db/src/schema/{auth,categories,products,settings}.ts`
- API routes: `artifacts/api-server/src/routes/{auth,storage,categories,products,settings}.ts`
- Admin-allowlist check: `artifacts/api-server/src/lib/adminAuth.ts` (single hardcoded owner email)
- Frontend pages: `artifacts/rine-beauty/src/pages/` (storefront `home.tsx`, `product-detail.tsx`; admin `admin/index.tsx` + `admin/{store-settings,categories-manager,products-manager}.tsx`)
- Cart/wishlist: client-only, `localStorage`-backed hooks in the frontend (no backend persistence — this is intentional, checkout is a WhatsApp handoff, not a payment flow)

## Architecture decisions

- Rebuilt from an original Firebase/Firestore/Cloudinary static site onto the monorepo's own stack: Postgres+Drizzle instead of Firestore, Replit Object Storage instead of Cloudinary, Replit Auth instead of Firebase Google Sign-In.
- Checkout is a WhatsApp deep link (`wa.me`), not a payment integration — matches the original store's real sales workflow (owner closes every sale personally over WhatsApp).
- Admin access is enforced server-side via a hardcoded single-email allowlist (`adminAuth.ts`), not a DB role — there is exactly one store owner.
- `Product.costPrice` lives in the same schema/response as public fields; the frontend simply never renders it on the public storefront. No separate "admin product" type.
- Seed data has no product images (object storage upload wasn't exercised for seeding) — this is acceptable for first build per project convention; real images get added by the owner through the admin dashboard's uploader.

## Product

- **Storefront (`/`)**: category filter, search, product grid, product detail page, cart + wishlist (persisted client-side), WhatsApp checkout with a prefilled order summary.
- **Admin dashboard (`/admin`)**: gated behind Replit Auth login, restricted to the store owner's email. Manages store branding/settings, categories, and full product CRUD (including an owner-only cost-price field for margin tracking).

## User preferences

_None recorded yet._

## Gotchas

- If you add `format: email` or `format: uri` to a string field in `lib/api-spec/openapi.yaml`, Orval v8 emits zod v4-only syntax (`zod.email()`, `zod.url()`) that breaks under the workspace's pinned zod v3 — avoid those formats, or re-check codegen output before relying on it.
- New `lib/*-web` packages (e.g. copied from a skill template) need `"composite": true` and `"declarationMap": true` in their `tsconfig.json`, or referencing packages fail `tsc --build` with TS6306.
- `numeric(...)` Drizzle columns default to returning strings; use `{ mode: "number" }` when the generated Zod response schema expects `zod.number()`.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
