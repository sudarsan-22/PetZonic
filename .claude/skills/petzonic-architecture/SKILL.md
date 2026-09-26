---
name: petzonic-architecture
description: Authoritative architecture and ground-truth reference for the PetZonic multi-repo pet-commerce platform (petzonic-api, petzonic-web, petzonic-admin, petzonic-infra). Load this before reading, planning, or modifying any PetZonic code, before answering questions about its structure, and before trusting any PetZonic markdown documentation — several project docs and all three audit reports in PetZonic/docs/09-audit-reports/ are verifiably inaccurate in places, and this file records which.
---

# PetZonic — Project Knowledge Base

**Compiled:** 2026-09-19, from direct read-only inspection of the working tree.
**Last re-measured:** 2026-09-26 — every repo's git history 2026-09-11 → 09-23 was read and the
changes verified against source (scheduler, production boot guards, breeder verification, AI
multi-tab routing, pre-owned, pharmacy, breeder hub). Cross-repo change list with commit IDs:
`PetZonic/docs/CHANGELOG.md`. Proposed (not built) features: `PetZonic/docs/10-feature-designs/`.

> ⚠️ **This codebase is under active development and counts drift within a single day.**
> On 2026-09-20 the `breeders` module was created *during* a documentation session, changing
> the module count from 25 to 26 in the span of minutes. Always re-measure before quoting a
> number:
> `ls petzonic-api/src/modules | wc -l` · `grep -cE '^model ' petzonic-api/prisma/schema.prisma`
> · `ls -d petzonic-api/prisma/migrations/*/ | wc -l` · `find petzonic-web/src/app -name page.tsx | wc -l`
>
> **Snapshot 2026-09-26:** 26 modules · 36 routers under `/api/v1` · 274 OpenAPI endpoints ·
> 68 models · 47 enums · 29 migrations · 3 BullMQ queues · web 86 pages · admin 29 pages ·
> tests: api 47, web 99 + 7 Playwright, admin 0.
**Root:** `/home/sudarsan/Documents/petzonic` (NOT itself a git repo — it is a container holding independent repos). **As of 2026-09-26 the org `.github` repo and this `.claude/` folder sit inside `PetZonic/`** (untracked by the PetZonic repo), not at the root.

> **Re-verified 2026-09-26** against git history (commits 2026-09-11 → 09-23). Cross-repo change list: `PetZonic/docs/CHANGELOG.md`.

## How to read this document

- **[VERIFIED]** — I confirmed this by reading the file or running a count/grep against source. Trust it.
- **[REPORTED]** — Came from a subagent's file-level read; consistent with everything else but I did not personally re-open that file.
- **[ASSUMPTION]** — My inference. Check before relying on it.
- **[UNVERIFIED]** — Named but never inspected. Do not repeat as fact.

Line references are `path:line` as of 2026-09-19 (spot-checked 2026-09-26). Line numbers drift; **always re-open the file** rather than quoting them back to a user.

> **Read this first:** the three audit reports in `PetZonic/docs/09-audit-reports/` are **not reliable**. See §16. They misstate the database schema, the password hashing algorithm, queue configuration, and migration history. Do not cite them.

---

## 1. Project Overview

PetZonic is the world's first **pet ecosystem** combining several product lines in one platform:

- **Pet marketplace** (C2C/B2C living-pet listings by sellers, breeders) with escrow payments
- **E-commerce** (pet products, new catalog + peer-to-peer pre-owned gear with admin moderation)
- **Pet pharmacy** (medicine catalog, prescription vault, admin prescription review, Rx gating at checkout)
- **District Breeder Hub** (breeder profiles by district; verification admin-granted)
- **Services & vet telehealth** (booking, live WebRTC consultation rooms)
- **Pet insurance** (plans, policies, claims)
- **Education/LMS** (courses, lessons, vet Q&A, feeding calculator)
- **Community** (forum posts, replies, votes, lost & found)
- **AI concierge** (conversational search across all domains with multi-tab routing, via local Ollama or Google Gemini)

Currency INR, Razorpay payments, Indian logistics (Shiprocket/Delhivery), Indian SMS (MSG91/Twilio).

**Architecture shape [VERIFIED]:** client-server. One Express 5 modular-monolith API + two Next.js 16 frontends + PostgreSQL 16 + Redis 7. Not microservices.

---

## 2. Complete Repository Structure

```
/home/sudarsan/Documents/petzonic/          <- NOT a git repo
├── PetZonic/                    [git] docs-only repo (product + architecture specs)
├── petzonic-api/                [git] BACKEND — the core of the system
├── petzonic-web/                [git] customer + seller + provider frontend
├── petzonic-admin/              [git] admin/ops console
├── petzonic-infra/              [git] Docker, Terraform, monitoring, ops scripts
├── petzonic-customer-app/       [git] EMPTY STUB (README + 3 CI files only)
├── petzonic-seller-app/         [git] EMPTY STUB (README + 3 CI files only)
├── PetZonic/.github/             [git] org profile + community health files (moved inside PetZonic/ by 2026-09-26)
├── [relocated 2026-09-19 →] petzonic-infra/qa/                        Python QA harness, Sept 19 run
└── [relocated 2026-09-19 →] PetZonic/docs/09-audit-reports/*.md       audit reports, each with an accuracy notice (§16)
```

Git-tracked file counts [VERIFIED 2026-09-26]: api 304 · web 348 · infra 497 (incl. `qa/`) ·
PetZonic 126 · admin 75. (The 2026-09-19 figures counted untracked files too and are not comparable.)

---

## 3. Repository Boundaries

**All 8 are separate git repositories with separate remotes.** There is no monorepo tooling, no submodules, and nothing that coordinates the 8 checkouts — they are manually co-located in one folder. A change spanning API + web is **two commits in two repos**.

| Repo | Remote | Branch | Purpose |
|---|---|---|---|
| PetZonic | `sudarsan-22/PetZonic` | `main` | Product/architecture documentation only. No code. |
| petzonic-api | `petZonic/petzonic-api` | `develop` | REST API, WebSockets, AI, queues, Prisma schema |
| petzonic-web | `petZonic/petzonic-web` | `develop` | Customer storefront **+ seller portal + provider portal** |
| petzonic-admin | `petZonic/petzonic-admin` | `develop` | Admin console |
| petzonic-infra | `petZonic/petzonic-infra` | `develop` | Compose topologies, Terraform, monitoring, backup/restore |
| petzonic-customer-app | `petZonic/petzonic-customer-app` | `develop` | **Stub.** Intended Flutter app; zero Dart exists. |
| petzonic-seller-app | `petZonic/petzonic-seller-app` | `develop` | **Stub.** Zero Dart exists. |
| .github | org `.github` | `main` | Org profile README, CODEOWNERS, issue/PR templates |

Each code repo has both `develop` and `main` locally; active work is on `develop`. Latest commits (2026-09-26): api 09-23, web 09-22, PetZonic 09-22, admin 09-20, infra 09-19. On 2026-09-26 the working trees held **uncommitted doc updates** (PetZonic, api, web, admin, infra, .github) from a doc-sync session — check `git status` before assuming clean.

**Relationships:** both frontends depend on `petzonic-api` over HTTP/WebSocket. **Nothing depends on the frontends.** `petzonic-infra` builds the other three via relative paths (`../../petzonic-api`, etc.), so its compose files assume the sibling-directory layout above. The two stub repos have no dependents and contain nothing.

> **Trap:** the existence of `petzonic-seller-app` misleads people into thinking seller features live elsewhere. **The seller portal is in `petzonic-web` at `/seller/*`.** Same for the provider portal at `/provider/*`.

---

## 4. Technology Stack

[VERIFIED] against source and `package.json`, not docs.

**Languages:** TypeScript (all three code repos), SQL (migrations), HCL (Terraform), Bash (ops scripts), Python (`petzonic-infra/qa/` cross-service harness only).

**Backend** — Node 22, Express 5, TypeScript, **Prisma 7** using the new `prisma-client` generator (output `src/generated/prisma`, *not* `@prisma/client`), PostgreSQL 16 + `pg_trgm`, Redis 7 (`ioredis`), BullMQ, Socket.IO 4, Zod, `jsonwebtoken` (HS256), `bcryptjs`, Pino, Helmet, Multer, `prom-client`, `swagger-ui-express`.

**Frontends** — Next.js 16 (App Router, `output: "standalone"`), React 19, Tailwind v4, Redux Toolkit, **Axios (NOT RTK Query)**, `socket.io-client`, `react-hook-form` + `zodResolver` (web), Zod.

**Package manager:** npm everywhere (`package-lock.json` is the committed lockfile). [VERIFIED] `petzonic-api/.github/workflows/backend-ci.yml` header explicitly records that earlier workflows wrongly assumed pnpm — **do not introduce pnpm/yarn.**

**Build:** `tsc` (API, with a separate `tsconfig.build.json`), `next build` (frontends), Docker multi-stage, Terraform ≥1.5 / AWS provider ~5.0.

**Explicitly NOT in the stack despite documentation claiming otherwise:** Flutter, NestJS, Meilisearch, MongoDB, Sentry, CloudWatch agent, FCM, Argon2, PostGIS, `petzonic-shared` package.

---

## 5. Frontend Architecture

### 5.1 petzonic-web (customer + seller + provider) — port 3001 docker / 3000 dev

[VERIFIED 2026-09-26] 303 files under `src/`, **86 pages**, 99 colocated test files. App Router, **no route groups** — plain nested folders.

**Guards are applied via scoped layouts** [VERIFIED 2026-09-26]:
- `AuthGuard`: `account/`, `chat/`, `checkout/`, `provider/`, `education/vet-consultation/`, `insurance/claims/`, `insurance/policies/`, `learn/my-courses/` layouts
- `SellerGuard`: `seller/layout.tsx`
- SEO-only layouts (metadata, no guard, added 2026-09-22): `breeders`, `contact`, `faq`, `insurance`, `pets`, `pharmacy`, `products`, `services`
- `src/components/auth/AuthGuard.tsx` self-documents as *"a UX guard, not a security boundary"*

**Routes** (grouped; all under `src/app/`):
- Commerce: `/`, `/products`, `/products/[slug]`, `/products/compare`, `/cart`, `/checkout`, `/order-success`, `/brands`, `/brands/[slug]`, `/pre-owned`, `/search`
- Pets: `/pets` (filters incl. `sellerType` ALL/BREEDER/SHOP/INDIVIDUAL), `/pets/[slug]`, **`/breeders`** (district hub, 2026-09-20), `/breeds`, `/breeds/[breed]` (breed hub, 2026-09-19), `/sellers/[id]`
- Pharmacy: **`/pharmacy`, `/pharmacy/products/[slug]`** (2026-09-20)
- Services/telehealth: `/services`, `/services/[id]`, `/services/consultation/[id]` (live WebRTC), `/education/vet-consultation{,/book,/[id]}`
- Insurance: `/insurance{,/[id],/calculator,/compare,/claims,/claims/[id],/policies,/policies/[id]}`
- Learn: `/learn{,/[slug],/courses,/courses/[id],/my-courses,/feeding-calculator,/vet-qa}`
- Community/chat: `/community`, `/community/[id]`, `/chat`, `/chat/[id]`
- Auth: `/auth` (single page for sign-in/up), `/auth/reset-password`
- Account: `/account{,/orders,/orders/[id],/bookings,/bookings/[id],/addresses,/payments,/wishlist,/reviews,/notifications,/notifications/preferences,/kyc,/settings,/support,/support/[id],/prescriptions}` (`/account/prescriptions` = prescription vault, 2026-09-20)
- Seller: `/sell`, `/seller{,/listings,/listings/new,/listings/[id]/edit,/products,/products/new,/orders,/payouts,/bank-account}`
- Provider: `/provider{,/register,/bookings,/consultations,/schedule,/services}`
- Static: `/about`, `/careers`, `/contact`, `/faq`, `/privacy`, `/terms`, `/refund-policy`

**State management** — `src/store/store.ts`: **exactly 4 slices** — `cart`, `wishlist`, `addressBook`, `auth`. Everything else is per-page `useState` + direct API call. Uses a per-request `makeStore()` factory (correct App Router pattern — do not convert to a module singleton). `src/store/StoreProvider.tsx` hydrates from storage in `useLayoutEffect` before first paint, then `store.subscribe`s to persist back. It also runs a **live role sync** on tab focus so a KYC approval grants the seller role without re-login.

**API client** — `src/lib/api.ts`. Single Axios instance, `withCredentials: true`. Notable: `getApiBaseUrl()` re-resolves the base URL **per request** to support LAN IP, ngrok and Cloudflare tunnels; response interceptor performs a **deduplicated** silent 401-refresh (`refreshInFlight`) and retries once via a `_retried` flag; `onAuthExpired` callback is wired into Redux by `StoreProvider`. **25** domain modules `src/lib/*Api.ts` (incl. `breedersApi`, `pharmacyApi`, `cartApi`, `addressesApi`). In Docker, `next.config.ts` rewrites proxy `/api/v1/*` and `/uploads/*` to `INTERNAL_BACKEND_URL`. Shared envelope helpers `unwrap()` / `unwrapPaginated()`.

**Offline/fallback data layer** — `src/data/*.ts` holds typed fixtures used both as fallback when the backend is unreachable and as unit-test fixtures.

**Real-time** — two *independent* Socket.IO connections: `src/lib/chatSocket.ts` (path `/chat`, module singleton) and inline in `src/components/consultation/VetConsultationRoom.tsx` (path `/consultation`, WebRTC signaling).

**Shared components** (`src/components/`, feature-grouped, not atomic): `auth/` (AuthGuard, SellerGuard, GoogleSignInButton), `cards/`, `consultation/VetConsultationRoom.tsx`, `discovery/AiShoppingAssistant.tsx`, `forms/`, `layout/` (Navbar, Footer, DashboardHeader, SellerHeader, ProviderHeader), `notifications/NotificationBell`, `orders/PayNowButton`, `seo/JsonLd`.

**Navigation** [VERIFIED 2026-09-26] — `Navbar.tsx` row 2: Pets · Local Breeders · Pre-Owned · Local Services · Pet Products · Brands · Pharmacy · Insurance · Community · Learn · Sell/Seller. **Breed Guides and Vet Care are only in `AllServicesDrawer.tsx`** (removed from row 2 on 2026-09-19 / 09-18).

**Homepage honesty change (2026-09-22, `c41557f`)** — fabricated testimonials and the App Store / Google Play banner were **removed**; replaced by a "Why buyers trust PetZonic" section stating real protections. `DemoNotice.tsx` and `favicon.ico` were deleted (`src/app/icon.svg` now). PWA manifest in `src/app/manifest.ts`; **no service worker**. Do not reintroduce fake social proof or app-store links.

> **Before editing petzonic-web, read `petzonic-web/AGENTS.md`.** It warns this Next.js 16 diverges from common knowledge and directs you to `node_modules/next/dist/docs/`.

### 5.2 petzonic-admin — port 3002 dev / 3000 docker

[VERIFIED 2026-09-26] **29 routes** (29 `page.tsx` files), all flat except `pharmacy/prescriptions{,/[id]}`. **One** root layout chaining `StoreProvider → IdleTimeoutProvider → AdminGuard → AdminShell`. No `middleware.ts`.

Screens: `/` (dashboard), `/login`, `/users`, `/kyc`, `/listings`, `/moderation`, `/products`, `/categories`, `/brands`, `/orders`, `/payouts`, `/payouts/[sellerId]`, `/disputes`, `/disputes/[id]`, `/providers`, `/reviews`, `/support`, `/support/[id]`, `/community`, `/education`, `/insurance`, `/promotions`, `/banners`, `/send-notification`, `/revenue`, `/audit-log`, `/settings`, **`/pharmacy/prescriptions`**, **`/pharmacy/prescriptions/[id]`** (split-screen verification, 2026-09-20). `/products` includes pre-owned moderation (condition + category-eligibility columns, 2026-09-17).

**State: exactly ONE slice (`auth`).** All 29 pages use local state. No shared table/modal/pagination components — each page reimplements loading/error/empty inline. `src/lib/adminApi.ts` is the largest client (~533 lines).

Nav defined centrally in `src/components/layout/AdminSidebar.tsx` — 4 groups (Overview, Trust & Safety, Marketplace Ops, Marketing & Platform) with live badge counts: `pendingModeration`, `pendingPrescriptions`, `pendingKyc`, `openDisputes`. Theme: full orange/amber PetZonic brand, Poppins/Inter (2026-09-18). `src/lib/adminPharmacyApi.ts` added.

**30-minute idle timeout** in `src/components/auth/IdleTimeoutProvider.tsx` → redirects to `/login?reason=idle_timeout`.

### 5.3 Customer / Seller mobile apps

**They do not exist.** Both repos contain only a README and 3 CI workflow files. [VERIFIED] zero Dart files. The CI workflows reference Flutter and would fail. Do not look for mobile code; do not "fix" these repos without an explicit decision.

---

## 6. Backend Architecture

**Entry:** `src/server.ts` → creates HTTP server, attaches `attachChatGateway` + `attachConsultationGateway`, calls `initializeQueues()`, listens on `config.port` (4000), handles SIGTERM/SIGINT → `closeQueues()`.

**App:** `src/app.ts` (~308 lines). Middleware order matters:
```
helmet → cors → morgan → express.json({limit:"10mb"}, captures rawBody)
  → urlencoded → cookieParser → csrfProtection (inline, ~:101-148)
  → metricsMiddleware → /uploads static → health+metrics → Swagger(/api/docs)
  → ~30 routers under /api/v1 → errorHandler (LAST)
```
`rawBody` is captured specifically so Razorpay webhook HMAC can be verified against the exact bytes. **Do not remove it or reorder `express.json` relative to the webhook routes.**

**26 modules** [VERIFIED 2026-09-26] in `src/modules/`: admin, ai-discovery, auth, banners, brands, **breeders**, cart, chat, community, docs, education, insurance, media, metrics, newsletter, notifications, orders, payments, pets, **pharmacy**, products, promotions, reviews, services, support, users.

**Newest two modules** (both added 2026-09-19/20; unchanged count since):
- **`pharmacy`** — fully wired: `/api/v1/pharmacy` (catalog, search, prescriptions upload &
  consultation-link, customer pet profiles) and `/api/v1/admin/pharmacy` (prescription review,
  verify, stats). Web pages at `/pharmacy`, `/pharmacy/products/[slug]`, `/account/prescriptions`; admin pages at
  `/pharmacy/prescriptions` and `/pharmacy/prescriptions/[id]`. Files: router/controller/service/repository/schema + admin router/controller + test.
- **`breeders`** — full stack as of 2026-09-20 01:20. Mounted at `/api/v1/breeders`
  (`GET /`, `GET /districts`, `GET /me`, `PUT /me`, `GET /:id`). Frontend at
  `petzonic-web/src/app/breeders/page.tsx` with client `src/lib/breedersApi.ts` — district
  browse, per-district breeder counts, `?district=` filtering.
  This is the feature closing the founder's core gap (buyers can't find breeders, only shops).
  [VERIFIED 2026-09-26] `BreederProfile.isVerified` now defaults to **`false`** (migration
  `20260922010000_breeder_verification_admin_granted` reset all profiles), but **no admin
  endpoint or screen grants it**, so no breeder is shown as verified. No `/breeders/[id]` detail
  page and no self-service profile form exist yet, though the API supports both.

**Per-module 4-tier pattern:** `*.router.ts` → `*.controller.ts` → `*.service.ts` → `*.repository.ts`, plus `*.schema.ts` (Zod). Admin surfaces live in separate routers (`admin-products.router.ts`, `admin-reviews.router.ts`, `admin-categories.router.ts`, `admin-banners.router.ts`, `admin-insurance.router.ts`, `admin-promotions.router.ts`, `admin-notifications.router.ts`, `admin-services.router.ts`).

- **Routers** are thin: path → middleware chain (`authenticate`, `authorize(...)`, rate limiter) → controller method.
- **Controllers** parse/validate, call service, `sendSuccess(res, ...)`, `next(err)` on failure.
- **Services** hold business rules and transactions.
- **Repositories** own Prisma calls.

**Shared libs** `src/lib/`: `prisma.ts`, `redis.ts`, `queue.ts`, `logger.ts`, `metrics.ts`, `s3.ts`, `upload.ts`, response helpers, validation.
**Middleware** `src/middleware/`: `auth.ts`, `error-handler.ts`, `rate-limit.ts`, `metrics.middleware.ts`.
**Config** `src/config/index.ts` — single env-driven config object + `validateConfig()`. **Production boot guard (2026-09-22, `c3bfaeb`)** [VERIFIED]: with `NODE_ENV=production`, `validateConfig()` throws if `JWT_SECRET` is in the known-insecure set or shorter than 32 chars, if `RAZORPAY_WEBHOOK_SECRET` is empty, or if `RAZORPAY_MOCK_ENABLED` is true. Outside production the fallback secret is the clearly named `DEV_JWT_SECRET`. `ALLOW_INSECURE_SECRETS` no longer exists.

**Other recent API behaviour** [VERIFIED 2026-09-26]:
- `GET /orders/:id/invoice` (2026-09-18) — JSON, or HTML with `?format=html` / `Accept: text/html`; buyer or admin only.
- Pre-owned products (2026-09-17) — `POST /products/pre-owned`, `GET /products/my-pre-owned`, `PATCH|DELETE /products/pre-owned/:id`; admin approve/reject. Only categories with `supportsPreOwned`; `GOOD`/`FAIR` require defects text.
- Pet listing query now accepts species attributes (`birdType`, `mutation`, `color`, `size`, `ageGroup`), `district`, `sellerType`, `verifiedOnly`; `limit` default 12.
- Admin user endpoints use an explicit `select` — `passwordHash`/`tokenVersion` never leave the API.
- Payout processing only includes orders with captured payment **and** `escrowStatus = RELEASED`.
- Refunds work for COD/mock orders without a `Payment` row.
- AI concierge (2026-09-23, `1328b48`) — `ConciergeIntentType` = PRODUCT, PET, BREEDER, SERVICE, PRE_OWNED, PHARMACY, BRAND, INSURANCE, LOST_FOUND, PET_CARE_ADVICE, EMERGENCY, MIXED; responses carry `targetTab` and per-domain result arrays; zero-result relaxation is domain-isolated. `AI_DISCOVERY_ROLLOUT_PERCENTAGE` code default 100.
- CORS: outside production (or localhost `CLIENT_URL`) the allowlist adds localhost:3000–3002, `*.petzonic.com`, ngrok and trycloudflare domains.

**Mount-order subtlety** [VERIFIED comment at `app.ts:291-296`]: `admin.router.ts` is mounted **last** among `/api/v1/admin/*` so the specific admin sub-routers win over its catch-all. Preserve this ordering.

**Background jobs** — `src/lib/queue.ts`: **three** BullMQ queues, `petzonic-email-queue` (concurrency 5, 3× exponential retry), `petzonic-broadcast-queue` (concurrency 2) and `petzonic-maintenance-queue` (added 2026-09-22, concurrency 1, retention 50/200). [VERIFIED] retention is `removeOnComplete: 100 / removeOnFail: 500` (email, :65-66) and `50 / 200` (broadcast, :87-88). **A scheduler exists since 2026-09-22** [VERIFIED]: the maintenance queue uses `upsertJobScheduler` for `auto-release-escrows` (`0 * * * *`, calls `autoReleaseExpiredEscrows()`) and `expire-abandoned-orders` (`*/15 * * * *`, cancels `PENDING_PAYMENT` orders older than 30 min and restores stock/listings). Jobs need Redis.

**Realtime** — `src/modules/chat/chat.gateway.ts` (path `/chat`; JWT handshake auth, room membership checked against `Conversation`, persists via Prisma, emits `new_message`) and `src/modules/services/consultation.gateway.ts` (path `/consultation`; WebRTC offer/answer/ICE relay).

**House convention — degrade gracefully, never fake success** [VERIFIED across modules]: Redis absent → in-memory rate limiter + direct (unqueued) execution; S3 absent → local disk; Razorpay absent → mock mode (development only — production boot now refuses mock); LLM absent → rule-based fallback or 503; **push provider is a permanent stub that reports itself unconfigured so the outbox marks `SKIPPED` rather than pretending to deliver** (`src/modules/notifications/providers.ts`). Follow this pattern in new integrations.

---

## 7. Database Architecture

**PostgreSQL 16 + Prisma.** Schema: `petzonic-api/prisma/schema.prisma` (~2076 lines).

[VERIFIED 2026-09-26] **68 models. 47 enums. 29 migrations. 79 `@@index` directives.**
(Was 61/26/66 on 2026-09-19 — the pharmacy and breeder migrations landed in between; the 29th migration, 2026-09-22, changed a default only.)

**Seven models added since the original survey**: `BreederProfile`, `OrderPrescription`,
`PharmacyProduct`, `PharmacySellerProfile`, `Prescription`, `PrescriptionItem`,
`UserPetProfile`.

**Three newest migrations**: `20260919195000_pet_pharmacy_models`,
`20260920011500_breeder_profile_and_district` (also adds a nullable
`district VARCHAR(100)` to `pet_listings` plus index `(district, status)`), and
`20260922010000_breeder_verification_admin_granted` (`breeder_profiles.is_verified` default
`false` + reset all rows to `false`).

Migrations run `20260823164725_init` → `20260922010000_breeder_verification_admin_granted`. They are phase-ordered (auth → pets → cart/orders → payments/escrow → chat → reviews → notifications → services → community → education → insurance → admin → newsletter/coupons/banners → pg_trgm search → pre-owned products → schema sync).

**Full model list** [VERIFIED]:
`Address, AuditLog, Banner, Booking, BreederProfile, Cart, CartItem, ContentLike, Conversation, Coupon, Course, CourseContent, CourseEnrollment, CourseProgress, DeviceToken, Dispute, EducationContent, InsuranceClaim, InsurancePartner, InsurancePlan, InsurancePolicy, KycSubmission, LostFoundPost, Message, NewsletterSubscriber, Notification, NotificationOutbox, NotificationPreference, Order, OrderItem, OrderPrescription, OtpCode, PasswordResetToken, Payment, Payout, PetBreed, PetListing, PetReport, PetSpecies, PharmacyProduct, PharmacySellerProfile, PlatformSettings, Post, PostFollow, PostVote, Prescription, PrescriptionItem, Product, ProductBrand, ProductCategory, ProviderSchedule, ProviderService, RefreshToken, Reply, ReplyVote, ReturnRequest, Review, ReviewHelpfulVote, ReviewReport, SellerBankAccount, ServiceProvider, SupportTicket, SupportTicketMessage, User, UserPetProfile, UserRole, VetConsultation, VetQA`

**Key relationships:**
- `User` 1:N `UserRole` (**many-to-many roles** — a user can be BUYER and SELLER at once), 1:N `Address`, `Order`, `RefreshToken`, …
- `PetSpecies` → `PetBreed` → `PetListing` → `PetReport`
- `ProductCategory` / `ProductBrand` → `Product` (dual-mode: `NEW` catalog vs `PRE_OWNED` peer listings with their own moderation workflow and optional `sellerId`)
- `Cart` → `CartItem`; `Order` → `OrderItem`, `ReturnRequest`, `Payment`
- `ServiceProvider` → `ProviderService`, `ProviderSchedule`, `Booking`
- `InsurancePartner` → `InsurancePlan` → `InsurancePolicy` → `InsuranceClaim`
- `Course` → `CourseContent` → `CourseEnrollment` → `CourseProgress`
- `Post` → `PostVote`, `PostFollow`, `Reply` → `ReplyVote`

**Important enums** [VERIFIED]:
- `Role`: `BUYER, SELLER, BREEDER, ADMIN` — **only these four**
- `PetListingStatus`: `DRAFT, PENDING_REVIEW, ACTIVE, PAUSED, SOLD, EXPIRED, REJECTED` — there is **no `PENDING_SALE`**
- `EscrowStatus`: `HELD, RELEASED, DISPUTED, REFUNDED` (field lives on **`Order`**, not `Payment`)
- `ProductCondition`: `NEW, PRE_OWNED`; `PreOwnedCondition`: `LIKE_NEW, GOOD, FAIR`; `PreOwnedStatus`: `PENDING_REVIEW, ACTIVE, PENDING_SALE, SOLD, REJECTED` (note: `PENDING_SALE` exists here, **not** on pet listings)
- `PrescriptionStatus`: `PENDING, APPROVED, REJECTED, EXPIRED, EXHAUSTED`; `DrugSchedule`: `OTC, SCHEDULE_H, SCHEDULE_H1, SCHEDULE_X, GENERAL_HEALTH`

**Constraints & indexes:**
- [VERIFIED] `Booking` has `@@unique([providerId, startTime])` — DB-level double-booking prevention. **Do not remove.**
- [VERIFIED] `Payment.razorpayPaymentId` is `@unique` — payment-capture idempotency.
- [VERIFIED] **Exactly ONE CHECK constraint exists**: `products_stock_non_negative_check CHECK (stock >= 0)`, added in `20260918150000_schema_sync/migration.sql:132`. The audit reports claim five CHECK constraints added by a migration `20260910143000_production_hardening_invariants` — **that migration does not exist in the repo** (§16 D-CHK).
- `Payment` has `refundedAmount Decimal @default(0)`.

**The schema file is the best documentation in this project.** It carries extensive prose comments explaining *why* gaps exist. Read it before trusting any markdown.

**Transaction idioms to copy:**
- Guarded test-and-set: `updateMany({ where: {id, status: "ACTIVE"}, ... })` then check `count === 0` → conflict. Used for pet purchase (`orders.service.ts:114-116`), refresh-token rotation, password-reset consumption.
- Atomic counters: `stock: { decrement: n }` / `{ increment: n }` inside `$transaction`.

---

## 8. Authentication & Authorization

**This is the most mature part of the codebase. Do not "simplify" it.**

**Passwords** [VERIFIED] — `bcryptjs`, **cost 12** (`src/modules/auth/auth.service.ts:61, 364, 395`). OTP codes use cost 10 (`src/modules/auth/otp.service.ts:22`). **Argon2 is not used anywhere** — absent from source and `package.json`, despite the hardening report claiming "Argon2id" (D3, §16).

**Login hardening** — constant-time `DUMMY_HASH` bcrypt compare when the user does not exist (blocks user enumeration via timing); `WEAK_PASSWORDS` denylist enforced on register/reset/change.

**Access token** — JWT HS256, ~15 min, `issuer: "petzonic-api"` / `audience: "petzonic-client"` pinned (RFC 8725), payload `{sub, userId, roles, tokenVersion}`. Issued in `src/modules/auth/token.service.ts`.

**Refresh token** — opaque `crypto.randomBytes(32)`, stored **SHA-256 hashed** in `RefreshToken`, delivered as an **HttpOnly cookie**. Rotation is an atomic guarded `updateMany` on `revokedAt: null` so exactly one concurrent request wins. **Replay of an already-revoked token revokes the entire token family** via `familyId`/`parentTokenId`/`replacedByTokenId` lineage (`token.service.ts:60-142`).

**Session revocation** — `logoutAll` / `changePassword` / `resetPassword` revoke all refresh tokens **and** increment `User.tokenVersion`, which invalidates in-flight access tokens immediately.

**Authorization — single enforcement point** [VERIFIED] `src/middleware/auth.ts`:
- `authenticate` — verifies Bearer JWT, attaches `req.user`
- `optionalAuthenticate` — never rejects; re-checks live DB status/tokenVersion so a revoked session degrades to anonymous
- **`authorize(...roles)` — re-fetches the user from Postgres on EVERY call**, re-checks `status === ACTIVE` and `tokenVersion`, then checks role membership. This is why bans and logout-all take effect instantly instead of at token expiry. It is intentional and costs a query per privileged request. **Do not "optimize" it into a pure JWT claim check.**

**Admin portal isolation is enforced server-side** [VERIFIED]: storefront `login`, OTP verify, and Google auth all **reject** users holding `ADMIN` (403 `ADMIN_PORTAL_REQUIRED`). `POST /api/v1/auth/admin/login` is the only admin path and requires the role.

**CSRF dual defense** (`app.ts:101-148`) — applies only to **cookie-authenticated state-changing** requests. Requires Origin/Referer on the CORS allowlist **plus** a custom header (`X-Requested-With` / `X-PetZonic-CSRF`). Bearer-only requests are exempt (correct — not CSRF-able).

**Roles** — `BUYER, SELLER, BREEDER, ADMIN` only. **Service Provider is NOT a role** — it is a separate `ServiceProvider` row with `PENDING/APPROVED/REJECTED` status. `SUPER_ADMIN` and `MODERATOR` are documented in admin repo docs but **do not exist** in code (§16).

**Frontend vs backend enforcement — critical distinction:**
- `petzonic-web/src/components/auth/AuthGuard.tsx` and `SellerGuard.tsx` → **UX only**, self-documented as not a security boundary
- `petzonic-admin/src/components/auth/AdminGuard.tsx` → client-side `roles.includes("ADMIN")`; admin has **no `middleware.ts`**
- **All real enforcement is `authorize(...)` in the API.** This is acceptable *because* the API enforces independently. When adding any privileged endpoint, the guard MUST be on the API route — a frontend check is never sufficient.

**Client token storage** [VERIFIED `petzonic-web/src/lib/authStorage.ts:75-90`]: the **access token IS persisted** to `localStorage` (rememberMe) or `sessionStorage` under key `petzonic:auth`, zod-validated on load. The **refresh token is correctly never written** to web storage (explicit comment at :76). Note this deviates from `PetZonic/docs/.../security.md`, which claims the access token is memory-only.

---

## 9. Major Business Workflows

**Customer product purchase**
`/products` → `/cart` (Redux + localStorage, server-synced) → `/checkout` (AuthGuard) → `POST /orders` → `POST /payments/create-order` (Razorpay) → Razorpay modal → `POST /payments/webhook` (HMAC over `rawBody`) → order CAPTURED → `/order-success`.
**All totals, tax, shipping and coupon discounts are computed server-side.** Client-supplied prices are ignored.

**Living-pet purchase (escrow)** — the highest-risk flow:
1. `orders.service.ts:114-116` atomically transitions the listing `ACTIVE → PAUSED` via guarded `updateMany`; `count === 0` → conflict. Prevents double-sale.
2. `payment.service.ts:114` sets `Order.escrowStatus = "HELD"` when the order contains a pet item.
3. Buyer calls `POST /orders/:id/confirm-receipt` → `orders.service.ts:264` calls `releaseEscrowIfHeld()` (`payment.service.ts:142`) → `RELEASED`.
4. Cancellation → `orders.repository.ts:62 cancelOrderInTransaction` restores stock, reverts the pet to `ACTIVE`, decrements coupon usage.
5. **`autoReleaseExpiredEscrows()` runs hourly** from the maintenance queue since 2026-09-22 (before that it had zero call sites). Seller payouts include only orders with `escrowStatus = RELEASED`.

**Seller onboarding** — `/sell` → `/account/kyc` (creates `KycSubmission`) → admin approves at admin `/kyc` → `SELLER` role granted → web's live role-sync picks it up on tab focus without re-login → `/seller/listings/new` → listing enters `PENDING_REVIEW` → admin moderates → `ACTIVE` → `/seller/orders` → ship (manual tracking entry **or** logistics factory) → `/seller/payouts`.

**Provider / telehealth** — `/provider/register` → admin approves at `/providers` → provider defines `/provider/services` + `/provider/schedule` → customer books (DB unique constraint blocks slot races) → live WebRTC room `/services/consultation/[id]` → vet files notes/prescription → `POST /services/bookings/:id/consultation/complete`.

**Admin** — `POST /auth/admin/login` → dashboard with live badges → moderation (pet listings + pre-owned gear), KYC review, **prescription review**, dispute resolution, forced order status, refunds, payouts (released escrow only), platform settings, audit log. **No breeder-verification action exists.**

**Pharmacy purchase** — customer uploads a prescription (`/account/prescriptions`) → admin verifies at `/pharmacy/prescriptions/[id]` → checkout allows prescription-only items only with a verified, unexpired prescription (`OrderPrescription` link).

**Scheduled maintenance** (since 2026-09-22) — hourly `auto-release-escrows`; every 15 min `expire-abandoned-orders` (unpaid > 30 min → cancelled, stock and listings restored).

**AI concierge** — `POST /api/v1/discovery/chat` → prompt-injection sanitization → **medical-emergency detection bypasses the LLM entirely** and returns hardcoded safety guidance → rule-based fast path, else Ollama/Gemini intent extraction → **parameterized Prisma query** (the LLM never receives credentials, SQL capability, or PII) → Redis session with sliding TTL → results for the detected domain plus a `targetTab` the web client navigates to. Includes a BOLA guard: "my orders/pets" phrasing is blocked for anonymous users and hard-scoped to `params.userId` when authenticated.

---

## 10. API Map

Base `/api/v1`. Envelope `{ success, data, error, meta }`. Swagger UI at `/api/docs`.

**Chain pattern (all modules):** `router (authenticate + authorize + rateLimiter) → controller (parse, sendSuccess/next) → service (rules, $transaction) → repository (Prisma)`.

**Worked example** [VERIFIED `src/modules/orders/`]:
```
POST /api/v1/orders/:id/confirm-receipt
  orders.router.ts:41   authenticate → ordersController.confirmReceipt
  orders.controller.ts  parse params → ordersService
  orders.service.ts:264 → releaseEscrowIfHeld(order.id)
  payment.service.ts:142 → prisma Order.updateMany({where:{id, escrowStatus:"HELD"}})
```

**Endpoint groups** [VERIFIED 2026-09-26 for orders, payments, products, pharmacy, breeders, admin, ai-discovery; REPORTED for the rest]. OpenAPI spec synchronised 2026-09-22: **274 endpoints across 36 modules** (`src/modules/docs/swagger.json`).

| Group | Router | Notable endpoints |
|---|---|---|
| auth | `auth.router.ts` | register, login, **admin/login**, otp/send, otp/verify, google, refresh, logout, logout-all, change-password, forgot/reset-password, me, sync |
| users | `users.router.ts` | profile, addresses CRUD, kyc submit/get, me/payouts, DELETE /me, `:id/public` |
| pets | `pets.router.ts` | list, my-listings, species, breeds, filter-config, suggestions, `:id`, create, **ai-assist** (Gemini), similar, update, status, delete, boost, report |
| products | `products.router.ts` + `admin-products.router.ts` + `admin-categories.router.ts` | list, search, categories, compare, pre-owned CRUD, `:slug`; admin CRUD/approve/reject/stock |
| cart | `cart.router.ts` | get, add, update, delete item |
| orders | `orders.router.ts` | **logistics/webhook** (no auth), create, mine, seller, `:id`, ship, tracking, live-tracking, shipping-label, **invoice** (JSON/HTML), cancel, return, confirm-receipt |
| payments | `payments.router.ts` | create-order, verify, **webhook**, history, refund/:orderId, seller/earnings, seller/payouts, seller/bank-account |
| services | `services.router.ts` + `admin-services.router.ts` | providers list/detail/availability/book, bookings, consultation session+complete, provider self-registration; admin approve/reject |
| reviews | `reviews.router.ts` + `admin-reviews.router.ts` | create, list, mine, update, delete, helpful, report; admin respond |
| chat | `chat.router.ts` + `/chat` gateway | rooms, messages |
| notifications | + `admin-notifications.router.ts` | list, unread-count, mark-read, preferences, devices; admin broadcast |
| community | `community.router.ts` | posts CRUD/vote/follow/pin, replies, lost-found |
| education | `education.router.ts` | content, courses, enroll, progress, vet-consultation, vet-qa, feeding-calculator |
| insurance | + `admin-insurance.router.ts` | plans, compare, calculate-premium, policies, claims, recommend; admin partners/plans |
| ai-discovery | `ai-discovery.router.ts` | `POST /chat`, `GET|DELETE /session/:id`, `GET /metrics` (admin) — mounted at **both** `/discovery` and `/chat/discovery` |
| pharmacy | `pharmacy.router.ts` + `admin-pharmacy.router.ts` | products, products/:slug, search, categories, prescriptions (list, detail, upload), pets CRUD; admin prescriptions, verify, stats |
| breeders | `breeders.router.ts` | list, districts, me (GET/PUT), `:id` |
| admin | `admin.router.ts` (**mounted last**) | dashboard, users (suspend/ban/reinstate), kyc, listings, moderation/listings (approve/reject/flag), moderation/products (approve/reject), orders (status, refund), disputes, reports (revenue, payouts), payouts/process, payouts/:sellerId, settings, audit-log, notifications/broadcast |
| support, promotions, banners, brands, newsletter, media, docs, metrics | respective routers | tickets, coupon validate, banner CRUD+clicks, brands, subscribe, upload, swagger, prometheus |

**Health:** `/health`, `/api/health`, `/api/v1/health`, `/health/liveness`, `/health/readiness`. **Metrics:** `/metrics`.
[VERIFIED] Compose healthchecks use `/health/liveness`; standalone compose and Terraform ALB use `/api/health`. Both exist — inconsistent convention, not a bug.

**Rate limiting** — 13 named limiters in `src/middleware/rate-limit.ts`, Redis-backed with in-memory fallback, separate dev (~500) and prod (5–30/hr) limits.

---

## 11. Infrastructure

All in `petzonic-infra/`.

**Four compose topologies** in `Deployment container/`:
- `docker-compose.yml` + `docker-compose.dev.yml` — [VERIFIED] **near-duplicates, only comments differ**. 14-service dev stack. Hardcoded dev-only secrets inline (e.g. `JWT_SECRET: change-this-secret-before-any-real-use`). Services: postgres, redis, backend, ollama, frontend(3001), admin(3002), prometheus(9090), alertmanager(9093), loki(3100), promtail, grafana(3003), cadvisor(8080), node-exporter(9100).
- `docker-compose.prod.yml` — adds **nginx** (only internet-facing, :80), **pgbouncer** (transaction pooling), **two backend replicas**. All other ports bound `127.0.0.1`. `${VAR}` interpolation; `JWT_SECRET` required with no default. Since 2026-09-22 the API also refuses to boot here without `RAZORPAY_WEBHOOK_SECRET` or with mock payments.
- `docker-compose.standalone.yml` — pulls prebuilt GHCR images, fully env-parameterized. **Not covered by CI validation.**

**Nginx** (`Deployment container/nginx/nginx.conf`) — upstreams `backend_cluster` (round-robin backend-1/2, `max_fails=3`, `proxy_next_upstream` failover), `frontend_cluster`, `admin_cluster`. WebSocket upgrade on `/socket.io/`. `client_max_body_size 25M`. **Listens on port 80 only — no TLS block.**

**Terraform** (`terraform/`) — single flat module, AWS ap-south-1, `environment` is just an input variable (no dev/staging/prod dirs or workspaces). Resources: 3-tier VPC (public / private-app / private-data), ECS Fargate (api/web/worker task defs), RDS PG16 Multi-AZ (`pg_trgm` preloaded, `rds.force_ssl=1`, 14-day backups), ElastiCache Redis 7 (encrypted at rest + in transit), ALB (sticky sessions for WebSockets, routes `/api/*`, `/socket.io/*`, `/consultation/*` to API target group), S3 + CloudFront with OAC, WAFv2 (managed rules + SQLi + 2000 req/5min), KMS CMK with rotation, Secrets Manager `db_credentials`.
[VERIFIED gaps] remote S3 backend is **commented out** in `versions.tf`; autoscaling exists for `api` only, not `web`/`worker`.

**Monitoring** — Prometheus (10s scrape) with **19 alert rules** in `monitoring/prometheus/alerts.yml` (5xx rate, p95/p99 latency, PG connections/locks/slow queries/cache-hit, Redis, BullMQ failed/stuck, Ollama down, container/host CPU-mem-disk, backup failed/missing). Alertmanager with severity routing + inhibition — **both receivers have only commented-out webhooks, so nothing notifies a human.** Loki + Promtail log aggregation. Grafana with 2 provisioned dashboards (`petzonic-infrastructure-overview`, `petzonic-logs-explorer`). Config lives in `Deployment container/monitoring/`.

**Ops scripts** (`scripts/`): `backup-database.sh` (pg_dump → gzip -9 → optional GPG AES-256 when `BACKUP_PASSPHRASE` set → sha256 → writes a Prometheus textfile that feeds the backup alerts → prune by `RETENTION_DAYS`), `restore-database.sh` (checksum verify → confirm → terminate connections → restore), `infra-maintenance.sh`, `generate-secrets.sh`, `push-images.sh`, `export-images-bundle.sh`.

[VERIFIED] `petzonic-infra/backups/` contains real `.sql.gz` dumps on disk, **but `.gitignore` correctly excludes `backups/*.sql*`, `*.sha256`, `*.json`** — only `.gitkeep` is tracked. They are local artifacts, not committed.

**CI/CD** — one workflow per repo, all validate-only:
- `petzonic-api/.github/workflows/backend-ci.yml` — **strongest**: Postgres 16 + Redis 7 service containers, `prisma generate/validate/migrate deploy`, typecheck, lint, **full test suite**, build, plus a guard that no `*.test.js` leaked into `dist/`
- `petzonic-web/.github/workflows/web-ci.yml` — typecheck, lint, test, build
- `petzonic-admin/.github/workflows/admin-ci.yml` — typecheck, lint, build (**no tests exist**)
- `petzonic-infra/.github/workflows/validate.yml` — `terraform fmt/validate` + `docker compose config` on 3 of 4 compose files
- stub repos — Flutter analyze/test/build workflows against no code

**No repo has build/push/deploy automation.** Image publishing is manual via `scripts/push-images.sh`.

---

## 12. Testing

| Repo | Framework | Count | Location |
|---|---|---|---|
| petzonic-api | Vitest + Supertest | **47 `*.test.ts`** [VERIFIED 2026-09-26] | colocated in `src/modules/*/` |
| petzonic-web | Vitest + Testing Library + jsdom | **99 test files** [VERIFIED] | colocated beside each page/component |
| petzonic-web E2E | **Playwright** | 7 specs | `e2e/` — auth, checkout, courses, insurance-calculator, nav, smoke, vet-consultation |
| petzonic-admin | — | **0** [VERIFIED] | **No test infrastructure, no `test` script** |
| petzonic-infra | — | config validation only | CI |

**API test infra:** `src/test/global-setup.ts`, `setup.ts`, `helpers.ts`. Runs against a **separate `petzonic_test` database** via `cross-env`-overridden `DATABASE_URL`, truncated/reseeded each run — dev data is never touched. Dedicated security suites: `auth.security.test.ts`, `database-redis.security.test.ts`, `ai-concierge.security.test.ts`, plus a golden-intent suite for AI and `ai-discovery-tabs.test.ts` (2026-09-23). Payout/escrow and admin-select cases added 2026-09-22.

**Web test helper:** `src/test/renderWithProviders.tsx` builds a fresh store per test. Documented gotcha — seed via `dispatch`, not `preloadedState` (TS typing conflict).

**Extra QA scripts** (`petzonic-api/scripts/`): `qa-reliability-suite.ts`, `security-gate-blackbox-suite.ts`, `verify-env-modes.ts`, `verify-metrics.ts`.

**Cross-service QA harness** — `petzonic-infra/qa/` (relocated 2026-09-19 from an untracked `scratch/` at the working-directory root; now committed) holds a Python/Playwright staged harness (`test_stage3..15_*.py`) and `master_matrix.md` covering **108 pages** from a 2026-09-19 run. [VERIFIED] It records **19 FAIL pages** — mostly admin mobile-viewport failures (390px/320px) across ~17 admin screens, plus `/breeds` failing on every viewport. **Caveat:** the `Status` column is written into the stage JSONs, not computed; several rows read `Functional: FAIL` yet `Status: PASS` (PAGE-001, 006, 013, 016, 022, 023, 024). `compile_master_qa_report.py` only aggregates. Treat that matrix as indicative, not authoritative. See `petzonic-infra/qa/README.md`.

---

## 13. Security Architecture

**Implemented controls** [VERIFIED unless noted]: bcrypt-12 + constant-time dummy hash + weak-password denylist; SHA-256-hashed opaque refresh tokens with family theft detection; live DB re-authorization on every privileged call; server-side admin portal isolation; dual CSRF defense; 13 Redis-backed rate limiters; Zod validation at every boundary; parameterized Prisma throughout (no raw SQL); Helmet; extensive Pino redaction incl. medical fields (`diagnosis`, `allergies`, `medication`, `prescription`) and financial (`pan`, `gstin`, bank, card); `rawBody` preserved for HMAC webhook verification; `SellerBankAccount` stores **last-4 only** (schema comment states full numbers belong in a PCI vault); LLM never receives credentials/PII/SQL; BOLA guard in the AI path; WAF + KMS + private subnets in Terraform.

**Known concerns — verified, ranked:**

1. ~~Escrow auto-release never runs~~ — **fixed 2026-09-22** (hourly maintenance job).
2. ~~`JWT_SECRET` fallback only warns in production~~ — **fixed 2026-09-22**: production boot throws on a placeholder or <32-char `JWT_SECRET`, a missing `RAZORPAY_WEBHOOK_SECRET`, or `RAZORPAY_MOCK_ENABLED=true`.
3. **No TLS by default** — nginx has no 443 block; Terraform's HTTPS listener is conditional on `enable_ssl` (default false) with empty `certificate_arn`.
4. **Logistics webhook fails OPEN** — [VERIFIED `src/modules/orders/logistics/shiprocket.provider.ts:206-208`] `verifyWebhookSignature` returns `true` when `SHIPROCKET_WEBHOOK_SECRET` is unset (`// If no secret configured, accept in testing`). `POST /api/v1/orders/logistics/webhook` has no `authenticate` middleware (correct for a webhook) so the signature is the only gate. When configured it uses `timingSafeEqual` correctly. **Set the secret in any internet-reachable environment.**
5. **Terraform Secrets Manager is wired but unused** — the secret and the `secretsmanager:GetSecretValue` IAM grant exist, but no ECS task definition has a `secrets` block; `DATABASE_URL` with the password is interpolated into the plain `environment` array.
6. **No Terraform remote state** — state (containing the generated DB password) lands on local disk.
7. **Access token persisted to web storage** — see §8. XSS-exfiltratable; 15-min TTL limits blast radius.
8. **Alertmanager notifies nobody** — 19 rules, no receiver.
9. **Silent notification failure** — email/SMS/push degrade to no-ops with no production startup assertion, so OTP login can fail with the code visible only in server logs.
10. ~~`RAZORPAY_MOCK_ENABLED` could be true in production~~ — **fixed 2026-09-22** (boot refuses).
11. **Admin console has zero tests** and holds the most destructive capabilities.
12. **Breeder verification has no grant path** — `isVerified` is admin-owned by design but no endpoint/screen sets it, so the breeder hub cannot show verified breeders.

---

## 14. Environment Configuration

**Variable NAMES only. Never record values here or anywhere in the repo.**

**petzonic-api** (read across `src/`; re-listed 2026-09-26):
`NODE_ENV, PORT, JWT_SECRET, JWT_EXPIRES_IN, JWT_ACCESS_EXPIRES_IN, REFRESH_TOKEN_EXPIRES_IN_DAYS, OTP_EXPIRES_IN_SECONDS, OTP_MAX_ATTEMPTS, PASSWORD_RESET_EXPIRES_IN_MINUTES, CLIENT_URL, DATABASE_URL, RAZORPAY_KEY_ID, RAZORPAY_KEY_SECRET, RAZORPAY_WEBHOOK_SECRET, RAZORPAY_MOCK_ENABLED, GOOGLE_CLIENT_ID, GEMINI_API_KEY, GEMINI_MODEL, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION, AWS_S3_BUCKET, AWS_S3_FORCE_PATH_STYLE, S3_ENDPOINT, S3_PUBLIC_URL, REDIS_URL, AI_DISCOVERY_ENABLED, AI_DISCOVERY_PROVIDER, AI_DISCOVERY_ALLOW_FALLBACK, AI_DISCOVERY_ROLLOUT_PERCENTAGE, OLLAMA_BASE_URL, OLLAMA_MODEL, SMTP_HOST, SMTP_PORT, SMTP_USER, SMTP_PASS, SMTP_SECURE, EMAIL_FROM, MSG91_AUTH_KEY, MSG91_SENDER_ID, MSG91_TEMPLATE_ID, MSG91_DLT_TE_ID, TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, TWILIO_FROM_NUMBER, DELHIVERY_BASE_URL, DELHIVERY_TOKEN, SHIPROCKET_BASE_URL, SHIPROCKET_EMAIL, SHIPROCKET_PASSWORD, SHIPROCKET_TOKEN, SHIPROCKET_WEBHOOK_SECRET`. `INITIAL_ADMIN_EMAIL` / `INITIAL_ADMIN_PASSWORD` are read by `prisma/seed-baseline.ts`; `SEED_MOCK_DATA` / `APP_ENV` by `docker-entrypoint.sh` and compose. **`ALLOW_INSECURE_SECRETS` was removed on 2026-09-22.**

Seed scripts: `db:seed` = `db:seed:mock` (developer mock data), `db:seed:demo` (end-to-end demo), `db:seed:baseline` (production baseline only).

⚠️ `petzonic-api/.env.example` is **incomplete** — `SMTP_*`, `MSG91_*`, `TWILIO_*`, `DELHIVERY_*`, `SHIPROCKET_*`, `S3_ENDPOINT`, `S3_PUBLIC_URL`, `EMAIL_FROM` are read in code but undocumented there.

**petzonic-web:** `NEXT_PUBLIC_API_URL` (default `http://localhost:4000/api/v1`), `NEXT_PUBLIC_SITE_URL`, `NEXT_PUBLIC_GOOGLE_CLIENT_ID`, `NEXT_PUBLIC_RAZORPAY_KEY_ID`, `NEXT_PUBLIC_AI_DISCOVERY_ROLLOUT_PERCENTAGE`, `E2E_API_URL`, `INTERNAL_BACKEND_URL` (server-only, used by `next.config.ts` rewrites; default `http://backend:4000`), `PORT`. Its `.env.example` declares only the first.

**petzonic-admin:** `NEXT_PUBLIC_API_URL`, `PORT`, `NODE_ENV`. That is all.

**petzonic-infra:** compose prod uses `${VAR}` for `POSTGRES_USER/PASSWORD/DB, DATABASE_URL, JWT_SECRET, CLIENT_URL, RAZORPAY_*, GOOGLE_CLIENT_ID, AWS_*, SMTP_*, EMAIL_FROM, GEMINI_API_KEY, GRAFANA_ADMIN_USER, GRAFANA_ADMIN_PASSWORD`. Standalone uses `.env.standalone.example` (`REGISTRY, IMAGE_TAG, *_PORT, POSTGRES_*, JWT_SECRET, CLIENT_URL, ADMIN_URL, NEXT_PUBLIC_API_URL, RAZORPAY_MOCK_ENABLED, AI_DISCOVERY_*, OLLAMA_MODEL`). Backup scripts use `BACKUP_PASSPHRASE`, `RETENTION_DAYS`. **There is no `.env.prod.example`.**

**Note:** dev-only seeded demo credentials appear in several committed READMEs and in `petzonic-admin/src/app/login/page.tsx` as a click-to-fill button. They are inconsistent across docs. Never treat them as production secrets and never propagate them.

---

## 15. Important Documentation — and which to trust

**Tier 1 — authoritative:**
- `petzonic-api/prisma/schema.prisma` — **the single best source of truth.** Rich prose comments explain design decisions and deliberate gaps.
- The source code itself.
- `PetZonic/docs/CHANGELOG.md` (2026-09-26) — cross-repo list of every change 2026-09-11 → 09-23 with commit IDs, plus a "Removed or changed" table.
- `petzonic-web/docs/` — re-synced 2026-09-26 (`routing-page-inventory.md` lists all 86 pages; `changelog.md`, `data-layer.md`, `known-limitations.md`, `testing-strategy.md` updated; the old "no backend / demo only" text was removed).
- `petzonic-web/AGENTS.md` — read before editing web.
- `petzonic-admin/ARCHITECTURE.md` (repo root, **not** in `docs/`).
- `petzonic-infra/README.md` and `Deployment container/README.md` (port table fixed 2026-09-26: admin 3002, Grafana 3003).

**Tier 2 — mostly accurate, re-synced 2026-09-26:**
`PetZonic/docs/README.md`, `02-technical-architecture/*` (each has a dated "implementation" section at the top), `03-database-design/{schema,data-dictionary}.md`, `04-api-design/*.md` (the May-era API docs now open with a verified "Implementation status" route table — **trust that table over the older examples below it**), `05-ui-ux/{website-pages,screen-inventory,admin-panel-pages}.md` (section 0 = current state), `06-project-roadmap/*` (progress snapshot at top), `07-development-guide/*`, `01-product-requirements/{PRD,feature-list}.md` (post-freeze features recorded as CR-004…CR-011).

**Tier 3 — STALE, do not trust:**
- `PetZonic/docs/03-database-design/er-diagram.md` — dated 2026-05-28, **never updated**; describes an abandoned schema (`franchises`, `litters`, `breeder_parents`, `escrow_holds`, `seller_payouts`, `kyc_verifications`, `chat_rooms`, `user_profiles`). None of those tables exist.
- `PetZonic/docs/01-product-requirements/user-stories/*`, `competitor-analysis.md`, `05-ui-ux/stitch-prompts-website.md`, `petzonic-api/docs/phase-*.md` — historical planning records, not updated.
- `application-interconnection.md` §5 was replaced 2026-09-26 with a correct diagram; §1–§4 were already accurate. (One leftover: a node label "AdminHeader → /admin/*" in §3.)

**Tier 4 — UNRELIABLE, do not cite without cross-checking (see §16):** the three audit reports, now at `PetZonic/docs/09-audit-reports/` (relocated 2026-09-19 from untracked root-level files; each carries an in-file accuracy notice pointing back to this section).

---

## 16. Documentation vs Implementation Discrepancies

All [VERIFIED] by direct comparison.

| ID | Claim | Source | Reality |
|---|---|---|---|
| **D1** | Table titled "complete inventory of all **61 models**" listing `SellerProfile`, `Wishlist`, `Shipment`, `Refund`, `ServiceBooking`, `UserPet`, `PetHealthRecord`, `PetVaccination`, `PetMedication`, `Species`, `Breed`, `Brand`, `ProductVariant`, `ProductImage`, `ContactSubmission`, `CouponUsage`, `AdminNote`, … | `DATABASE_REDIS_SECURITY_AUDIT.md` §1 | Count correct (61). **All 30 names checked are ABSENT from `schema.prisma`.** The audit describes a schema that does not exist; everything derived from it (retention, cascade, access matrix) is unverified. |
| **D-CHK** | 5 CHECK constraints applied via migration `20260910143000_production_hardening_invariants` | `FINAL_PRODUCTION_HARDENING_REPORT.md` §2 | **That migration does not exist.** Only ONE CHECK constraint exists: `products_stock_non_negative_check`, in `20260918150000_schema_sync/migration.sql:132`. |
| **D2** | Two Flutter mobile apps with Riverpod/GoRouter/Dio + dedicated screen docs | `tech-stack.md` §2, `05-ui-ux/{customer,seller}-app-screens.md`, org README | Both repos hold a README + 3 CI files. Zero Dart. |
| **D3** | Password hashing "**Argon2id** / bcrypt" | `FINAL_PRODUCTION_HARDENING_REPORT.md` §5 | Argon2 appears nowhere. bcryptjs only. |
| **D4** | bcrypt "**Cost Factor: 10**" for passwords | `DATABASE_REDIS_SECURITY_AUDIT.md` §2–3 | Actual **12**. (10 is OTP codes only.) `security.md` correctly says 12. |
| **D5** | BullMQ retention `101` / `100` / `{count:1000,age:86400}`+`{5000,604800}` | all three audits | Actual: email **100/500**, broadcast **50/200**. Three docs, three wrong answers. |
| **D6** | Pet lifecycle `ACTIVE → PENDING_SALE` | `DATABASE_REDIS_SECURITY_AUDIT.md` §10.1 | No `PENDING_SALE` in `PetListingStatus`. Actual is `PAUSED`. |
| **D7** | Rate limiters "fail open" | DB audit §7 | Contradicted by hardening report §3 ("fail-closed"). Code uses in-memory fallback = effectively fail-open. |
| **D8** | Seller bank account "Encrypted via AES-256 in DB" | hardening §5 | Schema stores **last-4 only**; full number never persisted. |
| **D9** | Test totals 976 / 1,022 / 1,023 / 1,026 | docs README / 3 audits | Four numbers. Actual *files* (2026-09-26): 47 api + 99 web + 0 admin. |
| **D10** | "19 domain modules" / "23 backend modules" | `system-architecture.md` §3.3 (fixed 2026-09-26) / E2E audit §1 | **26** directories in `src/modules/` (2026-09-26). |
| **D11** | ER diagram with `franchises`, `litters`, `escrow_holds`, … | `er-diagram.md` | Never updated since 2026-05-28. Zero of those tables exist. |
| **D12** | "no Redis yet"; admin at `/admin/*` inside web | `application-interconnection.md` §5 | **Fixed in the doc 2026-09-26.** Redis used extensively; admin is a separate app with root-level routes. |
| **D13** | `schema.md` documents 58 models | `03-database-design/schema.md` | Schema has 61. Missing exactly `ProductBrand`, `SupportTicket`, `SupportTicketMessage`. |
| **D14** | Roles incl. Super Admin, Franchise Owner, Broker, Vet, Caretaker | `security.md` §3.1, docs README | `Role` = `BUYER, SELLER, BREEDER, ADMIN` only. |
| **D15** | Admin per-route roles (`/settings`→SUPER_ADMIN, `/community`→MODERATOR) | `petzonic-admin/docs/routing-page-inventory.md`, `ARCHITECTURE.md` (both corrected 2026-09-26) | Nothing enforces these. `AdminGuard` checks `"ADMIN"` only; sidebar shows all groups to all admins. `AdminTopBar.tsx` hardcodes the label "Super Admin". |
| **D16** | Validation via class-validator; monitoring via Sentry/CloudWatch/X-Ray | `system-architecture.md` §8–9 | Zod, not class-validator. Prometheus/Grafana/Loki; no Sentry/CloudWatch agent. |
| **D17** | "23 test files, no e2e suite" | `petzonic-web/docs/testing-strategy.md` (fixed 2026-09-26) | 99 test files + 7 Playwright specs. |
| **D23** | Web app is "a frontend demo with no backend"; forms show `DemoNotice` | `petzonic-web/docs/{README,forms-and-validation,coding-standards,getting-started}.md` (fixed 2026-09-26) | Connected to the API since 2026-09-17; `DemoNotice.tsx` deleted 2026-09-22. |
| **D24** | "No scheduler exists"; escrow auto-release never runs | older docs and earlier versions of this file | Scheduler added 2026-09-22 (`petzonic-maintenance-queue`). |
| **D18** | WebRTC listed as future roadmap | `petzonic-web/docs/known-limitations.md` | `VetConsultationRoom.tsx` already implements real P2P WebRTC (1:1, no SFU). |
| **D19** | Default admin demo password | org/infra READMEs vs `petzonic-admin/src/app/login/page.tsx` + `STANDALONE_DEPLOYMENT.md` | Three docs state three different dev passwords, none matching the code's own click-to-fill demo button. Values intentionally not reproduced here — read the files if needed. |
| **D20** | Admin port 3000, Grafana 3002 | `Deployment container/README.md`, org profile README (both fixed 2026-09-26) | Compose maps admin `3002:3000`, Grafana `3003:3000`. |
| **D21** | `petzonic-shared` npm package in repo list | `tech-stack.md` §12 | Does not exist. |
| **D22** | "Production Ready & Fully Tested", verdict PASS | docs README + 3 audits | `petzonic-infra/qa/` QA run records 19 failing pages; matrix marks rows PASS while `Functional` reads FAIL. |

---

## 17. Known Technical Debt

**Deliberate and documented** (recorded as prose comments in `schema.prisma` — this is the *good* kind; do not "fix" without a product decision):
- **`Payment.orderId` is a mandatory FK to `Order`.** Consequence: courses, vet consultations, insurance policies and service bookings **cannot link to a real payment**. Courses/consultations fail fast with `PAYMENT_PROVIDER_UNAVAILABLE` (schema ~:1237-1247); insurance policies are issued **with no payment gate** (~:1437-1449); `Booking.refundAmount` is computed but never refunds (~:845-849). Fixing any requires a schema decision that has been consciously deferred.
- No MFA/TOTP and no admin sub-roles — explicitly scoped out (~:1567-1582).
- `VetConsultation.videoRoomUrl` left null rather than fabricated.

**Accidental:**
- **Two parallel telehealth implementations** — `/services/consultation/[id]` (real WebRTC) vs `/education/vet-consultation/*` (external `videoRoomUrl` link, shows "Video calling isn't configured yet"). Not wired together. Prime consolidation candidate.
- **petzonic-admin contains a large copy-pasted consumer API surface** — `bookService`, `purchasePolicy`, `calculatePremium`, `askVetQuestion`, `sendOtp`, `loginWithGoogle`, etc. in `src/lib/*Api.ts`, never called by any admin page.
- `petzonic-admin/src/components/layout/AdminHeader.tsx` is a 4-line `return null` no-op still imported/rendered by `kyc/page.tsx` and `disputes/[id]/page.tsx`.
- **Broken links** [still present 2026-09-26]: `petzonic-admin/src/app/disputes/[id]/page.tsx:58` → `/admin/disputes` (no such route; it is `/disputes`); `petzonic-web/src/app/services/consultation/[id]/page.tsx:95` → `/auth/login` (route is `/auth`).
- **Real bug** [still present 2026-09-26]: `petzonic-web/src/app/services/consultation/[id]/page.tsx:63` — `useEffect` dep array references bare `status`, undefined in scope, silently resolving to global `window.status` (always `""`). Intended dep is `authStatus`; the effect never re-runs on auth change.
- `docker-compose.yml` / `docker-compose.dev.yml` are near-identical duplicates; both referenced in docs.
- Duplicate `swagger.json` in `petzonic-api/docs/` and `src/modules/docs/` — only the latter is served.
- Admin port confusion: 3000 (Docker/docs) vs 3002 (`npm run dev` + hardcoded UI badge) vs a hardcoded `http://localhost:3001` marketplace link in `AdminTopBar.tsx`.
- Zod is a declared admin dependency used in exactly one file (`authStorage.ts`); no admin form validates with it.

---

## 18. Known Incomplete Features

1. ~~Escrow auto-release scheduler~~ — **done 2026-09-22** (`petzonic-maintenance-queue`).
2. **Both Flutter mobile apps** — documented in depth, zero code.
3. **FCM push delivery** — fully modeled (`DeviceToken`, `NotificationOutbox`, preferences) but the provider is a permanent stub.
4. **Payment gating** for courses / consultations / insurance / bookings (§17).
5. **Admin RBAC tiers** (`SUPER_ADMIN`, `MODERATOR`) — documented, unimplemented.
6. **MFA/TOTP** — scoped out.
7. **Admin test suite** — nonexistent.
8. **Deploy automation** — no repo builds/pushes/deploys.
9. **Terraform environments** — no dev/staging/prod separation, no remote state.
10. **Autoscaling** for `web`/`worker` ECS services.
11. **TLS** — not configured by default anywhere.
12. **Alertmanager receivers** — no notification channel.
13. **19 QA-failing pages** — admin mobile viewports (~17 screens) + `/breeds` (all viewports) — as of the 2026-09-19 run; not re-run since.
14. **Breeder verification grant** — no admin endpoint or screen (API default is unverified).
15. **PWA offline/push** — manifest only; no service worker.
16. **Designed, not built** — pet registry (Pet ID, rings/microchips), verified shops + near-me, health passport, lost/stolen alerts, breeder score, health guarantee, city licence helper, adoption, brand insights: see `PetZonic/docs/10-feature-designs/pet-registry-and-verified-network/`.

---

## 19. Important Conventions

**Backend**
- Modules are **feature-sliced**, not layered: `src/modules/<domain>/<domain>.{router,controller,service,repository,schema}.ts`.
- Admin endpoints go in a **separate** `admin-<domain>.router.ts`, never mixed into the public router.
- Routers stay thin — middleware chain + controller reference only.
- Validation is **Zod at the controller/schema edge**. Never validate deep in services.
- All responses use the `{ success, data, error, meta }` envelope via `sendSuccess` / the central `errorHandler`. Throw typed `AppError`; never `res.status().json()` an error inline.
- Imports use **explicit `.js` extensions** (ESM + NodeNext), e.g. `from "../payments/payment.service.js"`.
- Prisma client is imported from the **generated path** `src/generated/prisma`, not `@prisma/client`.
- **Degrade gracefully, never fake success** (§6).
- Race-sensitive writes use the **guarded `updateMany` + check `count === 0`** idiom inside `$transaction`.

**Database**
- Model names PascalCase singular; DB columns/tables snake_case via `@map`/`@@map`.
- UUID primary keys (`@default(uuid()) @db.Uuid`).
- Money as `Decimal @db.Decimal(10,2)`.
- Every schema change ships as a Prisma migration under `prisma/migrations/<timestamp>_<snake_name>/`.
- Document non-obvious decisions as prose comments **in the schema**.

**Frontend (both apps)**
- App Router; feature-named folders; colocated `page.tsx`.
- Auth/role gating via a **scoped `layout.tsx` wrapping a Guard component**.
- API access only through `src/lib/*Api.ts` modules built on the shared Axios instance — never call `fetch`/`axios` directly from a component. *(Two known violations exist in the consultation page.)*
- Redux only for genuinely cross-cutting state (web: cart/wishlist/addressBook/auth; admin: auth). Page data uses local state.
- Components grouped by feature under `src/components/<feature>/`.
- Web forms: `react-hook-form` + `zodResolver`, schemas in `src/lib/validation/`.

**Testing**
- Tests colocate next to source as `*.test.ts(x)`.
- API tests run against `petzonic_test`, never dev data.
- E2E lives in `petzonic-web/e2e/` (Playwright).

**Git**
- Conventional Commits, consistently applied. Work on `develop`. One commit per repo — cross-repo changes are multiple commits.

**Honest UI** (web, since 2026-09-22): no fabricated testimonials, ratings, counts or app-store links; show trust badges only when the API says they are true.

**Scheduled work** goes on `petzonic-maintenance-queue` as a new `task` with a stable `upsertJobScheduler` id — do not add `setInterval` or node-cron.

**Notably absent convention:** there are **no `TODO`/`FIXME`/`HACK` comments anywhere in any `src/`** [VERIFIED]. Gaps are recorded as prose in the schema or in `docs/known-limitations.md`. **Match this — do not introduce TODO markers.**

---

## 20. Critical "DO NOT BREAK" Rules

**Architecture boundaries**
1. Do not merge the repos or add cross-repo imports. They deploy independently.
2. Do not move the seller or provider portal out of `petzonic-web` into the stub repos.
3. Keep the 4-tier module pattern. Do not put Prisma calls in controllers or HTTP concerns in services.
4. Preserve router mount order in `app.ts` — `admin.router.ts` must stay mounted **last** among `/api/v1/admin/*`.
5. Do not reorder `express.json` / `rawBody` capture relative to webhook routes — Razorpay HMAC depends on exact bytes.
6. `petzonic-infra` compose files build via relative sibling paths; do not relocate repo directories.

**Security**
7. **Every privileged endpoint must carry `authenticate` + `authorize(...)` on the API route.** Frontend guards are UX only and are documented as such.
8. Do not "optimize" `authorize()` into a pure JWT claim check — the live DB re-fetch is what makes bans and logout-all instant.
9. Do not weaken refresh-token rotation: opaque token, SHA-256 at rest, HttpOnly cookie, atomic guarded rotation, **family-wide revocation on replay**.
10. Never lower bcrypt cost below 12 for passwords.
11. Never write the refresh token to `localStorage`/`sessionStorage`/Redux.
12. Keep admin portal isolation — storefront auth paths must keep rejecting `ADMIN`.
13. Never log or commit secrets; keep the Pino redaction list intact (it covers medical and financial fields).
14. Never store full card data or full bank account numbers. `SellerBankAccount` is last-4 only.
15. Do not pass PII, credentials, or SQL capability to any LLM provider; keep the AI path's parameterized-query and BOLA guards.

**Data integrity**
16. Keep `Booking @@unique([providerId, startTime])` — it is the only double-booking defense.
17. Keep `Payment.razorpayPaymentId @unique` — payment idempotency.
18. Living-pet purchase **must** stay an atomic guarded transition (`where: { status: "ACTIVE" }` → `PAUSED`, conflict on `count === 0`). A non-atomic read-then-write reintroduces double-sale.
19. Order cancellation **must** restore product stock, revert the pet to `ACTIVE`, and decrement coupon usage — all in one transaction (`orders.repository.ts:62`).
20. All order totals, tax, shipping and discounts are computed **server-side**. Never trust client-supplied prices.
21. Never write a destructive migration against `orders`/`payments` — these are financial records.
22. Do not point tests at the dev database; API tests use `petzonic_test`.

**Cross-service**
23. API response envelope `{success, data, error, meta}` is a contract consumed by both frontends — changing it breaks both.
24. `Role` enum values are consumed by both frontends' guards; adding/renaming requires coordinated changes.
25. Socket.IO paths `/chat` and `/consultation` are hardcoded client-side.
26. Nginx routes `/api/`, `/socket.io/`, `/admin`; ALB routes `/api/*`, `/socket.io/*`, `/consultation/*`. New top-level path prefixes need proxy updates.

---

## 21. Current Project State

- Code repos on `develop`. Most recent commits: api (Sep 23, AI multi-tab) → web / PetZonic (Sep 22) → admin (Sep 20, pharmacy queue) → infra (Sep 19, QA harness) → mobile stubs (May 29). Uncommitted doc re-sync changes exist as of 2026-09-26.
- Functionally broad and largely working: 108 pages exercised by the Sept 19 QA run, 19 failing.
- Backend is the most mature and best-tested surface; admin is the least tested (zero tests) with the most destructive capabilities.
- Not deployed: Terraform has never been applied (no remote state, HTTPS off, secrets wiring incomplete); no CD pipeline exists.
- The project **presents itself as production-ready** in its docs and audit reports. Based on verified evidence it is **feature-complete-ish but not production-hardened** — TLS, secret management in Terraform, alert routing, and durable file storage remain open. The escrow scheduler and production config guards were closed on 2026-09-22.
- `corporate-ca/Zscaler-Root-CA.pem` in api/web/admin indicates builds are expected inside a Zscaler-proxied corporate network.

---

## 22. Known Unknowns

Not inspected. Do not assert anything about these without opening them first.

1. **Full bodies** of `orders`, `payments`, and `admin` controllers/services — routes, schema constraints and key functions were traced, but not every transaction body was read line by line.
2. **Migration SQL contents** — names and the single CHECK constraint were verified; the rest of the SQL was not read.
3. Whether the audits' other claimed P0 fixes landed — the pet-status and escrow-release paths were confirmed present; `cancelOrderInTransaction`'s body was not diffed against the audit's quoted code.
4. **`swagger.json`** — synchronised 2026-09-22 (274 endpoints) per commit `0b0a5ae`, but its contents were not diffed against the routers; two copies still exist.
5. **`petzonic-infra/qa/stage*_results.json`** — individual failure reasons behind the 19 FAILs were not read.
6. **`petzonic-api/docs/*.md`** — 11 phase reports identified but unread.
7. Parts of **`PetZonic/docs/`** not re-read: competitor-analysis, 5 user-story files, team-roles, coding-standards, git-workflow, customer/seller app screen docs, 36 Stitch design HTML files. (The rest was re-synced 2026-09-26.)
8. **Delhivery** provider implementation and whether its webhook verification matches Shiprocket's fail-open behavior.
9. **Runtime behavior** — nothing was executed. No server started, no test run, no container built, no Terraform plan. **Every statement in this document is static analysis.**
10. Whether the 5 CHECK constraints the hardening report claims were applied **directly to a live database** outside version control (which would be undocumented schema drift).

---

## Quick Orientation for a New Session

```
Ground truth, in order:  schema.prisma  >  source code  >  PetZonic/docs/CHANGELOG.md  >  per-repo docs/  >  PetZonic/docs  >  09-audit-reports/ (cross-check first, see §16)

Backend entry:   petzonic-api/src/server.ts → src/app.ts
Auth enforcement: petzonic-api/src/middleware/auth.ts  (authorize() = the real boundary)
Schema:          petzonic-api/prisma/schema.prisma  (68 models, 29 migrations as of 2026-09-26, read the comments)
Jobs:            petzonic-api/src/lib/queue.ts      (email, broadcast, maintenance scheduler)
Web routes:      petzonic-web/src/app/**           (86 pages, incl. /seller/* and /provider/*)
Admin routes:    petzonic-admin/src/app/**         (29 pages, 0 tests)
Infra:           petzonic-infra/Deployment container/  +  terraform/

Before editing web:   read petzonic-web/AGENTS.md
Before citing a doc:  check §15 tier and §16 discrepancy table
Before touching money/pets/bookings: read §20 rules 16-22
```
