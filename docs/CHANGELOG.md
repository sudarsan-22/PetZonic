# PetZonic — Implementation Changelog

> Built from the git history of every repo. Each entry names the repo and commit so it can be
> traced. Newest first.  
> **Covers**: 2026-09-11 → 2026-09-26 · **Compiled**: 2026-09-26

---

## Current state (measured 2026-09-26)

| Metric | Value |
|---|---|
| API modules (`petzonic-api/src/modules/`) | 26 |
| Routers mounted under `/api/v1` | 36 |
| Documented endpoints (OpenAPI 3.0.3, `src/modules/docs/swagger.json`) | 274 |
| Prisma models / enums | 68 / 47 |
| Migrations | 29 (latest `20260922010000_breeder_verification_admin_granted`) |
| BullMQ queues | 3 (email, broadcast, **maintenance**) |
| Web pages (`petzonic-web`) | 86 |
| Admin pages (`petzonic-admin`) | 29 |
| Test files | API 47 · web 99 + 7 Playwright specs · admin 0 |

---

## Added

### 2026-09-23 — AI concierge: multi-tab routing (api `1328b48`)
- Intent types are now `PRODUCT`, `PET`, `BREEDER`, `SERVICE`, `PRE_OWNED`, `PHARMACY`,
  `BRAND`, `INSURANCE`, `LOST_FOUND`, `PET_CARE_ADVICE`, `EMERGENCY`, `MIXED`.
- Responses carry a `targetTab` (`/products`, `/pets`, `/breeders?state=…`, `/services`,
  `/pre-owned`, `/pharmacy`, `/brands`, `/insurance`) and cross-domain result bundles
  (breeders, pharmacy, pre-owned, brands, insurance plans, lost-and-found).
- Zero-result relaxation is isolated per domain; incompatible filters are cleared when the
  conversation switches domain.

### 2026-09-22 — Production safety and scheduled jobs (api `c3bfaeb`)
- **Scheduler now exists.** New `petzonic-maintenance-queue` (BullMQ job schedulers):
  - `auto-release-escrows` — **hourly**; runs `autoReleaseExpiredEscrows()` (release 7+ days
    after delivery with no dispute). Previously this function had no call sites.
  - `expire-abandoned-orders` — **every 15 minutes**; cancels `PENDING_PAYMENT` orders older than
    **30 minutes** and returns stock and pet listings.
- **Production boot refuses unsafe config** (`src/config/index.ts`): throws if `JWT_SECRET` is a
  known placeholder or shorter than 32 characters, if `RAZORPAY_WEBHOOK_SECRET` is missing, or if
  `RAZORPAY_MOCK_ENABLED` is true. (Previously only a warning.)
- **Seller payouts** only include orders whose escrow is `RELEASED` (held, disputed and
  refunded pet sales are excluded).
- **Refunds** handle COD and mock-mode orders that have no gateway `Payment` row.
- **Admin user endpoints** return an explicit field list — `passwordHash` and `tokenVersion` are
  no longer sent to the admin console.
- **Breeder verification is an admin decision**: `BreederProfile.isVerified` defaults to `false`;
  migration reset every existing profile to unverified. *No admin endpoint or screen grants it
  yet.*

### 2026-09-22 — API docs and brand (api `0b0a5ae`, `118aa93`; web `e1053a6`; PetZonic `b1bc889`)
- OpenAPI spec synchronised: 274 endpoints across 36 modules; Swagger UI shows live stats.
- Brand tagline unified to **"World's First Pet Ecosystem"** across API docs, seeds, web and docs.

### 2026-09-22 — Web trust, SEO and contact (web `c41557f`, `a1b4924`)
- Homepage "Why buyers trust PetZonic" section: escrow on every pet purchase, KYC-checked sellers,
  dispute window on every order.
- Per-section SEO `layout.tsx` metadata (breeders, contact, FAQ, insurance, pets, pharmacy,
  products, services); SVG app icon; real contact details on `/contact`.
- Active-tab background on navbar row 2.

### 2026-09-20 — District Breeder Hub (api `357d1d4`; web `1fc8a78`, `ba15172`, `67285ff`)
- `BreederProfile` model; nullable `district` on `pet_listings` with `(district, status)` index.
- `/api/v1/breeders`: `GET /`, `GET /districts`, `GET /me`, `PUT /me`, `GET /:id`.
- Web `/breeders`: district browse with per-district counts, breeder badging on pet cards,
  seller-type filters (breeder / shop / individual), breeder trust card.
- Navbar row 2 re-aligned: **Pets · Local Breeders · Pre-Owned · Local Services · Pet Products ·
  Brands** (+ Pharmacy, Insurance, Community, Learn, Sell).

### 2026-09-20 — Pet Pharmacy (api `8e74e9e`; web `9c0d66d`; admin `1d46bff`)
- Models `PharmacyProduct`, `Prescription`, `PrescriptionItem`, `OrderPrescription`,
  `PharmacySellerProfile`, `UserPetProfile`.
- `/api/v1/pharmacy` (catalog, search, categories, prescription vault + upload, customer pet
  profiles) and `/api/v1/admin/pharmacy` (prescription queue, verify, stats).
- Checkout blocks prescription-only items without a verified prescription.
- Web `/pharmacy`, `/pharmacy/products/[slug]`, `/account/prescriptions`.
- Admin `/pharmacy/prescriptions` and `/pharmacy/prescriptions/[id]` (split-screen review),
  sidebar badge.

### 2026-09-19 — Breed guides and hero (web `f0ad07d`, `e56d488`, `a9fafa9`, `05e9b11`, `46ee847`)
- `/breeds` and `/breeds/[breed]` breed knowledge hub, linked from the All-services drawer,
  footer and pets catalog.
- Homepage hero redesigned around "World's First Pet Ecosystem".

### 2026-09-19 — Cross-service QA harness (infra `9a3ae68`)
- `petzonic-infra/qa/` Python/Playwright staged harness and `master_matrix.md` (108 pages).

### 2026-09-18 — Orders, PWA and network (api `b427874`, `9b6d264`, `c543932`; web `86b6414`, `6c2dc39`, `bededc0`)
- `GET /api/v1/orders/:id/invoice` — tax invoice as JSON, or printable HTML with
  `?format=html` / `Accept: text/html`. Web invoice buttons.
- Web PWA manifest (`src/app/manifest.ts`), Schema.org JSON-LD, address sync, server-backed cart.
- Next.js rewrites proxy `/api/v1/*` and `/uploads/*` to `INTERNAL_BACKEND_URL`; tunnel
  detection; CORS allows ngrok and Cloudflare tunnel domains.
- `20260918150000_schema_sync` migration (token version, product brands, support tickets, the
  only CHECK constraint `stock >= 0`).

### 2026-09-17/18 — Observability stack (api `bcade26`, `dd35b20`; infra `ea7c261`, `571335b`, `56008a4`, `a2d0b89`)
- `/metrics` (Prometheus), `/health`, `/health/liveness`, `/health/readiness`; HTTP latency and
  subsystem telemetry.
- Infra: Prometheus, Grafana (2 dashboards), Alertmanager (19 alert rules, receivers not yet wired),
  Loki + Promtail, cAdvisor, Node Exporter, Docker log rotation, TSDB limits, maintenance script.

### 2026-09-17/18 — Seeding and compose (api `6857706`, `d4eb5ba`; infra `426b9fc`, `a64bdfe`)
- Seed scripts: `db:seed` / `db:seed:mock` (developer mock data), `db:seed:demo` (end-to-end demo:
  buyer, seller, admin, orders, telehealth, insurance), `db:seed:baseline` (production baseline).
- Separate dev compose (with mock data) and prod compose (without); AI discovery rollout 100%.

### 2026-09-17 — Pre-owned pet gear (api `14bc7ad`, `b32f360`; web `23797f3`, `e7a9873`; admin `584722d`, `0329a43`)
- Products have `condition` (`NEW` / `PRE_OWNED`) and a peer-listing workflow
  (`PreOwnedStatus`: `PENDING_REVIEW`, `ACTIVE`, `PENDING_SALE`, `SOLD`, `REJECTED`).
- API: `POST /products/pre-owned`, `GET /products/my-pre-owned`, `PATCH|DELETE
  /products/pre-owned/:id`; admin `POST /admin/products/:id/approve|reject`.
- Dynamic species attributes on pet listings (`attributes` JSON, species-specific filters).
- Web `/pre-owned`, seller `/seller/products`, `/seller/products/new`, All-services drawer.
- Admin moderation interface with condition and category-eligibility columns.

### 2026-09-17 — Education and connectivity (api `6b7cf64`; web `ddc22f5`, `a119df8`; admin `4a6002c`)
- Vet selector when booking a consultation; providerId resolution for consultations.
- Web and admin resolve the API base URL per request (LAN IP, non-localhost hosts).

### 2026-09-16/18 — Admin console (admin `c16f2fe`, `fd232f4`, `519b285`)
- `petzonic-admin` initialised (Next.js 16, Redux Toolkit, CI).
- Full PetZonic orange/amber brand and Poppins/Inter typography across the console.

---

## Removed or changed

| Date | Repo / commit | What changed |
|---|---|---|
| 2026-09-22 | web `c41557f` | **Removed fake testimonials** ("What Pet Parents Say" with named reviewers) — replaced by factual trust guarantees |
| 2026-09-22 | web `c41557f` | **Removed "Get PetZonic on Your Phone"** App Store / Google Play banner — no mobile app exists |
| 2026-09-22 | web `c41557f` | Removed `DemoNotice` component, default Next.js assets and `favicon.ico` (replaced by `icon.svg`) |
| 2026-09-22 | api `c3bfaeb` | Breeders are **no longer auto-verified**; all existing badges reset |
| 2026-09-22 | api `c3bfaeb` | Insecure production config now **fails boot** instead of warning |
| 2026-09-19 | web `2a9e8cb` | **Breed Guides removed from navbar row 2** (kept in All-services drawer and footer) |
| 2026-09-18 | web `31dbc59` | **Vet Care moved** from navbar row 2 into the All-services drawer |
| 2026-09-17 | web `23797f3` | Removed stray `.github/build.yml`, `lint.yml`, `test.yml` (real workflow is `.github/workflows/web-ci.yml`) |

## Still open (not changed in this period)

- No admin endpoint/screen to grant breeder verification.
- Payment gating for courses, consultations, insurance policies and bookings (`Payment.orderId` is
  mandatory).
- TLS off by default; Terraform never applied; Alertmanager has no receivers.
- Mobile apps: repos are still empty stubs.
- Admin console has zero tests.
