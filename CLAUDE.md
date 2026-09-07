# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project shape

BookBuy is a static multi-page HTML/CSS/vanilla-JS frontend (`frontend/`) plus a separate Express/Postgres API (`backend/`) that phases in real auth, payments, and commerce behind the existing pages. The frontend is **not** being migrated to a framework — new backend features are wired into the existing `.html` files via `fetch`, not a rewrite.

- `frontend/*.html` — the 14 static pages (see README.md's table for what each one is). Plain HTML5/CSS3/vanilla JS, no build step, no bundler. All internal links between pages are bare filenames (e.g. `href="product.html"`), so they only resolve correctly when the pages stay siblings in the same directory.
- `backend/` — Express API. ESM (`"type": "module"` in package.json), session-based auth, raw `pg` for queries (no ORM).

## Commands

Frontend (`frontend/`): no build/lint/test tooling exists. Serve statically from inside that folder, e.g. `cd frontend && python3 -m http.server 3000` or VS Code Live Server, and open the page directly.

Backend (`backend/`):
```bash
npm install
cp .env.example .env        # fill in DATABASE_URL, SESSION_SECRET, Paystack keys
npm run db:migrate          # applies backend/db/*.sql in filename order, idempotent (create-if-not-exists)
npm run dev                 # node --watch src/index.js, port 4000
npm start                   # no watch
```
No test suite and no lint config currently exist in either half of the repo. Health check: `curl http://localhost:4000/api/health`.

## Backend architecture

**Roadmap phases (P0–P4), reflected directly in file/table naming** — `db/001_init.sql` is P0/P1 (users only), `002_orders.sql` is P2 (bare order/payment tracking), `003_commerce.sql` is P3 (products, vendors, cart, order_items, wallet), `004_*.sql` is a P3 follow-up. Migrations are plain numbered `.sql` files applied in sorted order by `src/db/migrate.js` — add new schema changes as a new `NNN_description.sql` file rather than editing an existing one.

**Auth**: session-based, not JWT. `express-session` + `connect-pg-simple` stores sessions in Postgres (table auto-created via `createTableIfMissing: true`, not in a migration file). Cookie name is `bb_sid`. Every frontend `fetch` call to the API must pass `credentials: 'include'` or the session cookie won't round-trip — see the `API_BASE` + `fetch` blocks in `checkout.html`, `product.html`, `dashboard.html` for the existing pattern. `src/middleware/requireAuth.js` exports `requireAuth` (401 if no session) and `requireRole(...roles)` (403 if session role not in list) — role is cached on `req.session.role` at login, not re-fetched per request.

**Money**: always stored and computed in kobo (NGN's smallest unit, `amount_kobo` / `price_kobo` columns) as `bigint`, never floats. Never trust a price/amount from the client — `POST /api/orders` re-prices every line item server-side from the `products` table; `POST /api/payments/initialize` reads the amount to charge from the already-created `orders` row, not the request body.

**Order → payment flow** (P3 ordering, changed from P2 — see `004_orders_payment_reference.sql`): cart (`cart_items`) → `POST /api/orders` creates a `pending` order + `order_items` from the cart within a single transaction, computing `amount_kobo` server-side (shipping cost is validated against a hardcoded allowlist in `orders.js`, not trusted freely) → `POST /api/payments/initialize` starts the Paystack transaction against that order's stored amount → **`POST /api/payments/webhook` is the actual source of truth for payment success**, not the client-side popup closing; it verifies Paystack's HMAC-SHA512 signature against `req.rawBody` (captured separately in `express.json({ verify })` in `src/index.js` since a re-serialized body may not byte-match) before marking the order `paid` and clearing the user's cart. The frontend polls `GET /api/payments/:reference` after the popup closes to learn whether the webhook has landed yet.

**Routers** (`src/routes/`) mount under `/api/<name>` in `src/index.js`: `auth`, `payments`, `products`, `cart`, `orders`. `cart` and `orders` apply `requireAuth` to the whole router (`router.use(requireAuth)`); `products` is public; `payments` mixes public (`/webhook`) and protected routes per-route.

**Deliberately not implemented yet**: vendor/admin routes (though `requireRole` exists for when they land), a discount/promo system (checkout intentionally sends no discount field since the server can't validate one).

## Frontend/backend integration

Each HTML page that talks to the API defines its own `const API_BASE = 'http://localhost:4000/api'` near the top of its `<script>` block (see `checkout.html`, `product.html`, `dashboard.html`) — there's no shared JS module. CORS is locked to a single origin via `CORS_ORIGIN` in `backend/.env` (defaults to `http://localhost:8765`), which must match whatever port the static pages are actually served from in dev, or the session cookie won't be accepted.
