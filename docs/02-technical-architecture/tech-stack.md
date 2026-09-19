# PetZonic — Tech Stack

> **Version**: 1.1.0 (FROZEN)  
> **Date**: May 31, 2026 · **Accuracy-checked against source**: 2026-09-20
>
> **What is actually in use:** every row marked ✅ below is verified present in the working
> tree. Rows marked 📋 are *planned decisions that have not been implemented* — no code for
> them exists in any repo. The Flutter mobile stack (§2) is entirely in the 📋 category: the
> `petzonic-customer-app` and `petzonic-seller-app` repos contain a README and CI workflows
> only, with zero Dart files. Treat §2 as a forward-looking decision record, not a description
> of the system.

---

## 1. Stack Summary

| | Layer | Technology | Version / Details |
|---|-------|-----------|-------------------|
| ✅ | **Web (Customer, Seller, Provider)** | Next.js (React 19) | 16.x (Turbopack, App Router) — `petzonic-web`, 85 pages |
| ✅ | **Web (Admin)** | Next.js (React 19) | 16.x — `petzonic-admin`, 29 pages |
| 📋 | **Mobile (Customer & Seller)** | Flutter (Dart) | 3.x — **planned, not started; no code exists** |
| ✅ | **Backend** | Node.js + Express | 22.x (Node) / 5.x (Express) — 25 modules, 35 mounted routers |
| ✅ | **Language** | TypeScript | 5.x / 6.x |
| ✅ | **ORM** | Prisma | 7.x (`prisma-client` generator, output `src/generated/prisma`) |
| ✅ | **Database** | PostgreSQL | 16 — 68 models, 47 enums, 28 migrations, pg_trgm |
| ✅ | **Cache, Limiter & Sessions** | Redis | 7.x (ioredis + rate-limit-redis) — degrades to in-memory when absent |
| ✅ | **Queues** | BullMQ | 2 queues: `petzonic-email-queue`, `petzonic-broadcast-queue`. No scheduler/cron exists. |
| ✅ | **Search** | PostgreSQL pg_trgm | Native trigram fuzzy matching |
| ✅ | **AI Assist & Discovery** | Ollama (local) + Google Gemini | Falls back to rule-based extraction when no LLM is reachable |
| ✅ | **Object Storage** | AWS S3 | `@aws-sdk/client-s3` with local `/uploads` fallback. Cloudflare R2 is not configured. |
| ✅ | **Real-time** | Socket.io | 4.x — two gateways: `/chat` and `/consultation` (WebRTC signalling) |
| ✅ | **Containerization** | Docker & Docker Compose | Multi-container dev & prod topologies in `petzonic-infra` |
| 📋 | **Load Balancer** | Nginx | Compose config exists in `petzonic-infra`; never deployed |

**Package manager**: npm everywhere (`package-lock.json` is the committed lockfile).
Do not introduce pnpm or yarn — earlier CI workflows wrongly assumed pnpm and had to be fixed.

**Explicitly NOT in the stack**, despite appearing in older drafts of these docs: Flutter (no
code), NestJS, Meilisearch, MongoDB, Sentry, CloudWatch agent, FCM, Argon2 (passwords use
bcryptjs cost 12), PostGIS, and a `petzonic-shared` package.

---

## 2. Frontend — Mobile (Flutter) 📋 PLANNED — NOT IMPLEMENTED

> **Nothing in this section is built.** `petzonic-customer-app` and `petzonic-seller-app`
> contain only a README and 3 CI workflow files each; there are zero Dart files, and the CI
> workflows would fail if run. This section records the *intended* mobile approach for when
> mobile work begins. Do not cite it as a description of the current system.

### Why Flutter?
| Criteria | Flutter | React Native | Native (Kotlin/Swift) |
|----------|---------|-------------|----------------------|
| Single codebase | ✅ iOS + Android + Web | ✅ iOS + Android | ❌ Separate codebases |
| UI Quality | ⭐⭐⭐⭐⭐ Pixel-perfect | ⭐⭐⭐⭐ Good but bridge | ⭐⭐⭐⭐⭐ Best |
| Performance | ⭐⭐⭐⭐⭐ Near-native (Skia/Impeller) | ⭐⭐⭐⭐ Bridge overhead | ⭐⭐⭐⭐⭐ Best |
| Dev Speed | ⭐⭐⭐⭐⭐ Hot reload, single lang | ⭐⭐⭐⭐ Hot reload, JS | ⭐⭐⭐ Slower (2 teams) |
| Long-term | ⭐⭐⭐⭐⭐ Google-backed, growing | ⭐⭐⭐⭐ Meta-backed | ⭐⭐⭐⭐⭐ Platform-native |
| India Community | ⭐⭐⭐⭐⭐ Huge Flutter dev pool | ⭐⭐⭐⭐ Large | ⭐⭐⭐ Smaller |
| Cost | 💰 1 team | 💰 1 team | 💰💰 2 teams |

**Decision**: Flutter — Best balance of performance, UI quality, single codebase, and developer availability in India.

### Flutter Key Libraries

| Purpose | Library |
|---------|---------|
| State Management | Riverpod 2.x |
| Navigation | GoRouter |
| HTTP Client | Dio |
| WebSocket | socket_io_client |
| Local Storage | Hive / SharedPreferences |
| Image Handling | cached_network_image |
| Forms | flutter_form_builder |
| Maps | google_maps_flutter |
| Push Notifications | firebase_messaging |
| Payment | razorpay_flutter |
| Camera | image_picker |
| Video | video_player |
| Animations | Lottie |
| Responsive | flutter_screenutil |
| Auth | firebase_auth (for phone OTP) |

### Mobile Architecture Pattern
- **Clean Architecture** (Presentation → Domain → Data layers)
- **Feature-first** folder structure
- **Repository pattern** for data access
- **Riverpod** for dependency injection and state

---

## 3. Frontend — Web (React.js + Next.js)

### Why React.js + Next.js?
| Criteria | Next.js | Angular | Nuxt (Vue) |
|----------|---------|---------|-----------|
| SEO (SSR/SSG) | ⭐⭐⭐⭐⭐ Built-in | ⭐⭐⭐ Angular Universal | ⭐⭐⭐⭐ Good |
| Performance | ⭐⭐⭐⭐⭐ ISR, Edge | ⭐⭐⭐⭐ Good | ⭐⭐⭐⭐ Good |
| E-commerce fit | ⭐⭐⭐⭐⭐ (Vercel Commerce, Shopify use it) | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Ecosystem | ⭐⭐⭐⭐⭐ React + massive libs | ⭐⭐⭐⭐ Complete framework | ⭐⭐⭐⭐ Vue ecosystem |
| Learning curve | ⭐⭐⭐⭐ Moderate | ⭐⭐⭐ Steeper | ⭐⭐⭐⭐⭐ Easy |
| Dev pool (India) | ⭐⭐⭐⭐⭐ Largest | ⭐⭐⭐⭐ Large | ⭐⭐⭐ Growing |

**Decision**: React.js with Next.js framework — Best for SEO-critical e-commerce, large React ecosystem, excellent performance features, largest developer pool.

### Web Key Libraries

| Purpose | Library |
|---------|---------|
| UI Components | Tailwind CSS + Headless UI |
| State Management | Redux |
| Forms | React Hook Form + Zod validation |
| HTTP | Axios |
| Real-time | socket.io-client |
| Payment | Razorpay JS SDK |
| Maps | @react-google-maps/api |
| Charts (Admin) | Recharts |
| Tables (Admin) | TanStack Table |
| Toast/Alerts | Sonner |
| Icons | Lucide React |
| Date handling | date-fns |
| Image optimization | Next.js Image (built-in) |

### Web Architecture
- **Server Components** for data-fetching pages (SEO)
- **Client Components** for interactive UI
- **API routes** for BFF (Backend for Frontend) pattern when needed
- **ISR** (Incremental Static Regeneration) for product/pet pages

---

## 4. Backend — Node.js + Express

### Why Node.js + Express?
| Criteria | Node.js + Express | NestJS | Django (Python) | Spring Boot (Java) |
|----------|-------------------|--------|-----------------|--------------------|
| TypeScript | ⭐⭐⭐⭐⭐ Full support | ⭐⭐⭐⭐⭐ First-class | ❌ Python | ❌ Java/Kotlin |
| Flexibility | ⭐⭐⭐⭐⭐ Full control | ⭐⭐⭐ Opinionated | ⭐⭐⭐ Opinionated | ⭐⭐⭐ Opinionated |
| Dev Speed | ⭐⭐⭐⭐⭐ Lightweight, fast | ⭐⭐⭐⭐ Boilerplate | ⭐⭐⭐⭐ Good | ⭐⭐⭐ Slower |
| Ecosystem | ⭐⭐⭐⭐⭐ Largest npm ecosystem | ⭐⭐⭐⭐ NestJS wrappers | ⭐⭐⭐⭐ Good | ⭐⭐⭐⭐ Good |
| Learning curve | ⭐⭐⭐⭐⭐ Minimal | ⭐⭐⭐ Decorators, DI | ⭐⭐⭐⭐ Moderate | ⭐⭐⭐ Steep |
| WebSocket | ⭐⭐⭐⭐⭐ socket.io native | ⭐⭐⭐⭐⭐ Built-in gateway | ⭐⭐⭐ Channels | ⭐⭐⭐⭐ |
| Shared lang with frontend | ✅ TypeScript | ✅ TypeScript | ❌ | ❌ |

**Decision**: Node.js + Express — Lightweight, full control over architecture, massive ecosystem, same language as frontend, simpler learning curve, direct library usage without framework wrappers.

### Backend Key Libraries

| Purpose | Library | Version / Notes |
|---------|---------|-----------------|
| ORM | Prisma | 7.x (Client + PG Adapter + Engine) |
| Validation | Zod | 4.x (Strict type schemas) |
| Auth & Security | jsonwebtoken + bcryptjs + helmet | Token rotation & password hashing |
| WebSocket | socket.io | 4.x (Real-time in-app chat) |
| Cache & Limits | ioredis + rate-limit-redis | Shared Redis rate limiter with fallback |
| AI Photo Assist | @google/genai | Gemini AI photo analysis & welfare check |
| Cloud Storage | @aws-sdk/client-s3 | AWS S3 / Cloudflare R2 with local disk fallback |
| File Upload | multer | Multi-part upload handler |
| Swagger Docs | swagger-ui-express | Interactive OpenAPI 3.0 UI (/api/docs) |
| Logging | pino + pino-pretty | Structured JSON production logs |
| Testing | Vitest + Supertest | Unit & Controller/Service integration tests |

### Backend Architecture Pattern
- **Modular Monolith** (19 cohesive domain modules in `src/modules/`)
- **Router → Controller → Service → Repository** 4-tier layer pattern
- **Thin Routers**: Pure route-to-controller mapping with auth & rate limit guards
- **Controllers**: Request validation, status code assignment, response formatting
- **Services**: Business logic, domain rules, third-party integrations
- **Repositories**: Isolated Prisma database queries and transactions
- **Zod schemas**: Schema-first input validation
- **Centralized Error Handling**: Standardized `{ success, data, error, meta }` envelope

---

## 5. Database — Single Unified PostgreSQL 16

### Why Unified PostgreSQL over MongoDB + PostgreSQL?
- **Relational Integrity & ACID**: E-commerce, escrow payments, user roles, orders, and pet listings require strict foreign keys, transactional atomicity, and consistent reads across all domains.
- **Native JSONB Flexibility**: PostgreSQL 16's binary JSON (`JSONB`) provides all the document-store benefits previously desired from MongoDB (e.g. flexible health info, specifications, user preferences) without running a second database cluster.
- **Zero Dual-Write Latency**: Operating a single unified database eliminates the need for distributed transactions, two-phase commits, or eventual consistency synchronization between Mongo and Postgres.
- **Reduced Infrastructure & Operational Overhead**: One database engine to backup, monitor, replicate, and scale in production.

### Why Prisma ORM?
- **Type Safety**: Automatically generates TypeScript definitions directly from the 58 Prisma models.
- **Declarative Migrations**: Version-controlled, reproducible SQL migration files.
- **Connection Pooling**: Native integration with PostgreSQL connection pools.
- **Single Source of Truth**: `prisma/schema.prisma` is the definitive data contract for the entire backend.

---

## 6. Search — PostgreSQL pg_trgm (Trigram Fuzzy Search)

### Why Native PostgreSQL pg_trgm over Meilisearch?
- **Out-of-the-Box Typo Tolerance**: `pg_trgm` provides similarity-based fuzzy matching (e.g. finding "Golden Retriever" even when queried as "Goldn Retrever").
- **Zero Additional Infrastructure**: Runs directly inside PostgreSQL without provisioning, maintaining, or paying for an external Meilisearch server.
- **Instant Real-Time Consistency**: When a pet or product is created or updated, it is immediately searchable with zero indexing lag or webhook sync delay.
- **GIN Indexing**: Trigram GIN indexes (`gin_trgm_ops`) provide fast sub-10ms query times for catalogs well beyond 100,000 listings.

---

## 7. Real-time — Socket.io

### Why Socket.io?
- **Fallback support**: WebSocket → HTTP long-polling (important for Indian networks)
- **Room-based**: Perfect for buyer-seller chat rooms
- **Scalable**: Redis adapter for multi-server support
- **Express integration**: Easy middleware setup with socket.io
- **Flutter client**: Official socket_io_client package
- **Proven**: Used by major chat platforms

---

## 8. Cloud — AWS

### Why AWS over GCP/Azure?
| Criteria | AWS | GCP | Azure |
|----------|-----|-----|-------|
| India region | ✅ Mumbai (ap-south-1) | ✅ Mumbai | ✅ Central India |
| Service breadth | ⭐⭐⭐⭐⭐ Most comprehensive | ⭐⭐⭐⭐ Good | ⭐⭐⭐⭐ Good |
| Cost (moderate budget) | ⭐⭐⭐⭐ Free tier generous | ⭐⭐⭐⭐ Competitive | ⭐⭐⭐ Pricier |
| Managed PostgreSQL | ✅ RDS | ✅ Cloud SQL | ✅ Azure DB |
| S3 equivalent | ✅ S3 (industry standard) | ✅ Cloud Storage | ✅ Blob Storage |
| Dev community (India) | ⭐⭐⭐⭐⭐ Largest | ⭐⭐⭐⭐ Growing | ⭐⭐⭐ Enterprise focus |
| Startup credits | ✅ Activate program | ✅ Startup program | ✅ |

**Decision**: AWS — Most comprehensive services, Mumbai region with low latency, largest community, generous free tier for initial development.

---

## 9. Payment — Razorpay

### Why Razorpay?
- **India-first**: UPI, all Indian banks, regional wallets
- **Split payments**: Built-in marketplace commission support (Route/Linked accounts)
- **Escrow**: Payment hold before release (critical for pet purchases)
- **Easy integration**: Good SDKs (JS, Flutter, Node.js)
- **Compliance**: PCI DSS Level 1, RBI compliant
- **Pricing**: 2% per transaction (competitive for India)
- **Support**: Indian time zone support

---

## 10. DevOps & Tooling

| Tool | Purpose |
|------|---------|
| **GitHub** | Code hosting, PRs, code review |
| **GitHub Actions** | CI/CD pipelines |
| **Docker** | Containerization for consistent environments |
| **Docker Compose** | Local development environment |
| **Terraform** | Infrastructure as Code (AWS resources) |
| **ESLint + Prettier** | Code quality and formatting |
| **Husky** | Git hooks (lint on commit, test on push) |
| **Commitlint** | Conventional commits enforced |
| **Sentry** | Error tracking and monitoring |
| **Swagger/OpenAPI** | API documentation (swagger-ui-express) |

---

## 11. Development Environment

```
Required installations:
- Node.js 22.x (via nvm)
- Flutter 3.x SDK
- Docker Desktop
- PostgreSQL 16 (via Docker)
- Redis 7 (via Docker)
- Ollama (via Docker / local binary for AI discovery)
- VS Code (with recommended extensions)
- Android Studio (for Android emulator)
- Xcode (for iOS simulator — Mac only)
```

### VS Code Extensions (Recommended)
- Flutter + Dart
- ESLint + Prettier
- Prisma
- REST Client (Thunder Client)
- Docker
- GitLens
- Tailwind CSS IntelliSense

---

## 12. Monorepo vs Multi-repo

**Decision: Multi-repo** (initially)

| Repo | Contents | Status |
|------|----------|--------|
| `petzonic-api` | Node.js + Express backend, Prisma schema | ✅ Active (`develop`) |
| `petzonic-web` | Next.js website — customer **+ seller + provider** portals | ✅ Active (`develop`) |
| `petzonic-admin` | Next.js admin panel | ✅ Active (`develop`) |
| `petzonic-infra` | Docker, Terraform, monitoring, QA harness | ✅ Active (`develop`) |
| `PetZonic` | This documentation (no code) | ✅ Active (`main`) |
| `.github` | Org profile, CODEOWNERS, issue/PR templates | ✅ Active (`main`) |
| `petzonic-customer-app` | Intended Flutter customer app | ⛔ Empty stub — README + CI only |
| `petzonic-seller-app` | Intended Flutter seller app | ⛔ Empty stub — README + CI only |
| `petzonic-shared` | Shared types/constants (npm package) | ⛔ **Does not exist** — never created |

There are **8 repositories**, each with its own remote and no monorepo tooling, submodules,
or workspace linking. A change spanning API + web is two commits in two repos.
`petzonic-infra` builds the others through relative paths (`../../petzonic-api`), so its
compose files assume all repos are checked out side by side in one parent directory.

> **Common trap**: the existence of `petzonic-seller-app` suggests seller features live
> elsewhere. They do not — the **seller portal is in `petzonic-web` at `/seller/*`**, and the
> provider portal at `/provider/*`.

**Rationale**: Simpler CI/CD, independent deployment cycles, smaller team can focus on one repo at a time. Can migrate to monorepo (Turborepo/Nx) later if needed.