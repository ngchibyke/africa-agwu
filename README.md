# LensLink

A photography & videography marketplace connecting clients with professional
photographers and videographers — discovery, portfolios, booking, in-app messaging,
secure payments via Paystack, provider payouts, and an admin dashboard.

Built for **NGCHIBYKE LTD**.

## Tech stack

- **Framework**: Next.js 16 (App Router, Turbopack, Server Actions)
- **Language**: TypeScript
- **Database**: SQLite via Prisma (local dev) — swap the datasource for PostgreSQL in production
- **Auth**: NextAuth v5 (credentials/email+password, JWT sessions, role-based access)
- **Styling**: Tailwind CSS v4
- **Payments**: Paystack (checkout, webhooks, transfers/payouts)
- **File storage**: local disk under `public/uploads` (swap for S3/Cloudinary/Supabase Storage in production)

## Getting started

```bash
npm install
cp .env.example .env   # then fill in real values, see below
npx prisma migrate dev # creates prisma/dev.db and applies the schema
npm run db:seed        # seeds demo providers, clients, bookings, reviews
npm run dev
```

Visit http://localhost:3000.

### Demo logins (from `npm run db:seed`)

| Role | Email | Password |
| --- | --- | --- |
| Admin | the value of `PLATFORM_SUPPORT_EMAIL` (defaults to `ngchibykefotos@gmail.com`) | `Admin123!` |
| Provider | `ada-eze@example.com` (and 5 more — see `prisma/seed.ts`) | `Password123!` |
| Client | `ngozi.umeh@example.com` (and 2 more) | `Password123!` |

## Environment variables

See `.env.example` for the full list with comments. Summary:

| Variable | Purpose |
| --- | --- |
| `DATABASE_URL` | Prisma connection string. `file:./dev.db` locally; a Postgres URL in production. |
| `AUTH_SECRET` | Session signing secret for NextAuth. Generate with `openssl rand -base64 32`. |
| `NEXTAUTH_URL` | Base URL of the app (used for auth callbacks). |
| `PAYSTACK_SECRET_KEY` | Server-side Paystack secret key. Never expose to the client. |
| `NEXT_PUBLIC_PAYSTACK_PUBLIC_KEY` | Paystack public key (safe to expose). Not currently used client-side since checkout is handled via Paystack's hosted page, but kept for future inline-checkout use. |
| `PLATFORM_SUPPORT_EMAIL` / `PLATFORM_SUPPORT_PHONE` | Shown in the footer and used as the seeded admin account. |
| `UPLOAD_DIR` | Local upload directory (MVP file storage). |

## Paystack setup

1. Create a Paystack account and grab your **test** secret/public keys from
   `https://dashboard.paystack.com/#/settings/developers`.
2. Put the secret key in `PAYSTACK_SECRET_KEY` and the public key in
   `NEXT_PUBLIC_PAYSTACK_PUBLIC_KEY`.
3. **Webhook**: in the Paystack dashboard, set the webhook URL to
   `https://<your-deployed-domain>/api/payments/webhook`. This endpoint verifies the
   `x-paystack-signature` header (HMAC-SHA512 of the raw body with your secret key) before
   trusting any event, per Paystack's documented webhook security model.
4. The booking payment flow:
   - Client clicks **Pay now** on an accepted booking → `initiatePaymentAction` creates a
     `Payment` row and calls `POST /transaction/initialize`, then redirects to Paystack's
     hosted checkout.
   - On completion, Paystack redirects back to `/api/payments/callback`, which re-verifies
     the transaction server-side (`GET /transaction/verify/:reference`) — the query string
     is never trusted directly.
   - The `/api/payments/webhook` endpoint is the **source of truth**: it confirms payment
     even if the client closes the tab before the redirect completes. Both paths call the
     same idempotent `markPaymentPaid()` helper.
   - On payment success, the booking moves to `CONFIRMED` and platform commission /
     provider earnings are computed from `PlatformSetting.commissionPercent` (default 10%,
     editable by admins under **Admin → Commission & settings**).
5. **Payouts**: providers add a bank account (verified via Paystack's `/bank/resolve` and
   turned into a `/transferrecipient`), then request withdrawals, which call Paystack's
   `/transfer` endpoint. This requires your Paystack account to have transfers enabled
   (may require additional KYC with Paystack for live keys).

No test-mode simulation of "Google automatically pays you for sign-ups" or similar is
implemented, per the spec's instruction — all platform revenue comes from booking
commissions, optional featured-listing/premium fees (configurable, not yet billed
automatically), and real, policy-compliant advertising if/when integrated.

## Project structure

```
prisma/schema.prisma        Database schema (Users, ProviderProfile, Booking, Payment, …)
prisma/seed.ts               Demo data seed script
src/lib/                     Shared server logic: auth, prisma client, Paystack client,
                              money formatting, storage, validation
src/app/                     App Router pages
  (public)                   /, /search, /providers/[slug], /login, /signup
  /account                   Client dashboard (bookings, notifications)
  /bookings/[id]              Booking detail: messaging, accept/decline, payment,
                              delivery upload, completion, reviews
  /provider/dashboard/*       Provider dashboard: overview, profile, portfolio,
                              services & packages, availability, bookings, earnings,
                              payouts
  /admin/*                    Admin dashboard: overview/analytics, users, providers
                              (verify/feature), bookings, reports & disputes,
                              categories, commission & platform settings
  /api/auth/*                  NextAuth + registration
  /api/payments/*               Paystack callback + webhook
```

Most mutations are Next.js **Server Actions** (colocated `actions.ts` files) rather than
separate REST routes — the two exceptions are `/api/payments/*` (must be reachable by
Paystack's servers) and `/api/auth/*` (NextAuth's own routes, plus a JSON `register`
endpoint kept for potential non-web clients).

## Assumptions made

The original spec ("MISSION" brief) left several implementation details unspecified.
Where ambiguous, the safest, most conventional choice was made rather than inventing
behaviour:

- **Currency**: Nigerian Naira (NGN) throughout, since the business (NGCHIBYKE LTD) and
  Paystack integration are Nigeria-based. All amounts are stored in kobo (minor units) to
  avoid floating-point rounding errors.
- **Local dev database**: SQLite, for zero-setup local development. The schema is
  provider-agnostic Prisma; switching `prisma/schema.prisma`'s datasource to
  `postgresql` and setting `DATABASE_URL` is the only change needed for a production
  Postgres database.
- **File storage**: MVP uploads (portfolio photos/videos, delivered files) are written to
  local disk under `public/uploads`. This does **not** survive redeploys on most hosting
  platforms (e.g. Vercel's ephemeral filesystem) — before going live, swap
  `src/lib/storage.ts` for a cloud bucket (S3, Cloudinary, Supabase Storage) behind the
  same `saveUploadedFile()` interface.
- **Commission model**: a single platform-wide commission percentage (admin-configurable,
  default 10%), deducted at payment time, with provider earnings only "released" (an
  `EARNING` transaction created) once the client confirms job completion — modelling an
  escrow-like flow without literally holding funds outside Paystack's settlement.
- **Featured listings / premium accounts**: the spec lists these as revenue streams and
  they're modelled in the schema and admin settings, but self-service billing for them
  (a provider paying to feature themselves) isn't wired up yet — today an admin manually
  toggles a provider's "Featured" flag. Automating this is a natural next step (a Paystack
  subscription/charge on the existing payment plumbing).
- **Advertising**: the schema includes an `Advertisement` model (placement, impressions,
  clicks) for a future ad-serving feature, but no ad network (e.g. Google AdMob) is wired
  up — the spec was explicit that fake/incentivized ad activity must never be simulated,
  so nothing here fabricates ad revenue.
- **Disputes**: a lightweight dispute/report flow exists (either party can be reported;
  admins resolve disputes and the booking moves to `COMPLETED`). Refunds through Paystack
  are not automated — the spec didn't define refund policy, so this intentionally routes
  to a human admin decision rather than guessing a refund percentage/policy.
- **Notifications**: in-app only (a `Notification` row + list in the client/provider
  dashboards). Email/SMS/push delivery isn't implemented — the spec asked for
  notifications without specifying a channel, and adding a transactional email/SMS
  provider is a config-only addition later (e.g. Resend, Termii) once you have accounts
  for those services.
- **Search**: text/location filtering plus specialty-category and rating filters,
  backed directly by SQL queries (`ILIKE`-style `contains` matching). No dedicated search
  index (e.g. Algolia/Meilisearch) — fine at MVP scale, worth revisiting if the provider
  catalog grows large.

## Deployment notes

1. Provision a PostgreSQL database and set `DATABASE_URL`; update
   `prisma/schema.prisma`'s `datasource.provider` to `"postgresql"`, then run
   `npx prisma migrate deploy`.
2. Set all secrets in `.env.example` as real environment variables on your host (never
   commit `.env`).
3. Point object storage (see "File storage" above) instead of local disk if deploying to
   a platform with an ephemeral filesystem.
4. Register the Paystack webhook URL against your production domain (see "Paystack
   setup" above) — this is required for payments to confirm reliably.
5. `npm run build && npm run start`, or deploy to a Next.js-compatible host (Vercel,
   Railway, Render, etc.).
