# PetZonic — Tech Stack

> **Version**: 1.0.0  
> **Date**: May 28, 2026

---

## 1. Stack Summary

| Layer | Technology | Version |
|-------|-----------|---------|
| **Mobile** | Flutter (Dart) | 3.x (latest stable) |
| **Web (Customer)** | Next.js (React 19) | 15.x |
| **Web (Admin)** | Next.js (React 19) | 15.x |
| **Backend** | NestJS (Node.js) | 11.x (NestJS) / 22.x (Node) |
| **Language** | TypeScript | 5.x |
| **ORM** | Prisma | 6.x |
| **Database** | PostgreSQL | 16 |
| **Cache** | Redis | 7.x |
| **Search** | Meilisearch | 1.x |
| **Queue** | Bull (Redis-backed) | 5.x |
| **Real-time** | Socket.io | 4.x |
| **Cloud** | AWS | - |
| **CI/CD** | GitHub Actions | - |
| **Containerization** | Docker | - |

---

## 2. Frontend — Mobile (Flutter)

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

## 3. Frontend — Web (Next.js)

### Why Next.js?
| Criteria | Next.js | Angular | Nuxt (Vue) |
|----------|---------|---------|-----------|
| SEO (SSR/SSG) | ⭐⭐⭐⭐⭐ Built-in | ⭐⭐⭐ Angular Universal | ⭐⭐⭐⭐ Good |
| Performance | ⭐⭐⭐⭐⭐ ISR, Edge | ⭐⭐⭐⭐ Good | ⭐⭐⭐⭐ Good |
| E-commerce fit | ⭐⭐⭐⭐⭐ (Vercel Commerce, Shopify use it) | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Ecosystem | ⭐⭐⭐⭐⭐ React + massive libs | ⭐⭐⭐⭐ Complete framework | ⭐⭐⭐⭐ Vue ecosystem |
| Learning curve | ⭐⭐⭐⭐ Moderate | ⭐⭐⭐ Steeper | ⭐⭐⭐⭐⭐ Easy |
| Dev pool (India) | ⭐⭐⭐⭐⭐ Largest | ⭐⭐⭐⭐ Large | ⭐⭐⭐ Growing |

**Decision**: Next.js — Best for SEO-critical e-commerce, large React ecosystem, excellent performance features, largest developer pool.

### Web Key Libraries

| Purpose | Library |
|---------|---------|
| UI Components | Tailwind CSS + Headless UI |
| State Management | Zustand (client) + React Query (server) |
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
- **App Router** (Next.js 15 app directory)
- **Server Components** for data-fetching pages (SEO)
- **Client Components** for interactive UI
- **API routes** for BFF (Backend for Frontend) pattern when needed
- **ISR** (Incremental Static Regeneration) for product/pet pages

---

## 4. Backend — NestJS

### Why NestJS?
| Criteria | NestJS | Express.js | Django (Python) | Spring Boot (Java) |
|----------|--------|-----------|-----------------|-------------------|
| TypeScript | ⭐⭐⭐⭐⭐ First-class | ⭐⭐⭐⭐ With setup | ❌ Python | ❌ Java/Kotlin |
| Architecture | ⭐⭐⭐⭐⭐ Opinionated, modular | ⭐⭐ Freestyle | ⭐⭐⭐⭐⭐ Opinionated | ⭐⭐⭐⭐⭐ Enterprise |
| Scalability | ⭐⭐⭐⭐⭐ Microservice-ready | ⭐⭐⭐ Manual | ⭐⭐⭐⭐ Good | ⭐⭐⭐⭐⭐ Excellent |
| Dev Speed | ⭐⭐⭐⭐⭐ CLI, decorators, DI | ⭐⭐⭐⭐ Fast but messy | ⭐⭐⭐⭐ Good | ⭐⭐⭐ Slower |
| Maintainability | ⭐⭐⭐⭐⭐ Enforced patterns | ⭐⭐ Depends on team | ⭐⭐⭐⭐ Good | ⭐⭐⭐⭐⭐ Excellent |
| WebSocket | ⭐⭐⭐⭐⭐ Built-in gateway | ⭐⭐⭐ socket.io manual | ⭐⭐⭐ Channels | ⭐⭐⭐⭐ |
| Shared lang with frontend | ✅ TypeScript | ✅ JavaScript | ❌ | ❌ |

**Decision**: NestJS — Enterprise-grade structure with TypeScript, same language as frontend, modular design supports future microservices migration, excellent WebSocket support.

### Backend Key Libraries

| Purpose | Library |
|---------|---------|
| ORM | Prisma 6 |
| Validation | class-validator + class-transformer |
| Auth | @nestjs/passport + @nestjs/jwt |
| WebSocket | @nestjs/websockets + socket.io |
| Queue | @nestjs/bull |
| Cache | @nestjs/cache-manager + ioredis |
| File Upload | multer + @nestjs/platform-express |
| Swagger | @nestjs/swagger |
| Config | @nestjs/config |
| Throttle | @nestjs/throttler |
| Health Check | @nestjs/terminus |
| Testing | Jest + Supertest |
| Logging | nestjs-pino (structured JSON logs) |
| Scheduling | @nestjs/schedule |

### Backend Architecture Pattern
- **Modular Monolith** (each feature is a NestJS module)
- **Controller → Service → Repository** pattern
- **DTOs** for request/response validation
- **Guards** for authentication & authorization
- **Interceptors** for response transformation, caching, logging
- **Pipes** for input validation
- **Filters** for exception handling

---

## 5. Database — PostgreSQL

### Why PostgreSQL?
| Criteria | PostgreSQL | MongoDB | MySQL |
|----------|-----------|---------|-------|
| ACID Compliance | ⭐⭐⭐⭐⭐ Full | ⭐⭐⭐ Eventually consistent | ⭐⭐⭐⭐⭐ Full |
| JSON Support | ⭐⭐⭐⭐⭐ JSONB, indexable | ⭐⭐⭐⭐⭐ Native | ⭐⭐⭐ Basic |
| Complex Queries | ⭐⭐⭐⭐⭐ Advanced SQL, CTEs | ⭐⭐⭐ Aggregation pipeline | ⭐⭐⭐⭐ Good |
| Relationships | ⭐⭐⭐⭐⭐ Foreign keys, joins | ⭐⭐ Manual references | ⭐⭐⭐⭐⭐ Good |
| Full-text Search | ⭐⭐⭐⭐ Built-in (basic) | ⭐⭐⭐ Atlas Search | ⭐⭐⭐ Basic |
| Scalability | ⭐⭐⭐⭐ Read replicas, partitioning | ⭐⭐⭐⭐⭐ Horizontal | ⭐⭐⭐⭐ Good |
| E-commerce fit | ⭐⭐⭐⭐⭐ Stripe, Shopify use it | ⭐⭐⭐ Flexible but risky | ⭐⭐⭐⭐ |

**Decision**: PostgreSQL — Relational integrity critical for e-commerce (orders, payments, inventory), ACID compliance for financial transactions, JSONB for flexible fields, proven at scale.

### Why Prisma (ORM)?
- **Type-safe**: Auto-generated TypeScript types from schema
- **Migration system**: Version-controlled schema changes
- **Query performance**: Optimized SQL generation
- **Developer experience**: Auto-complete, intuitive API
- **Schema-first**: Single source of truth for database structure

---

## 6. Search — Meilisearch

### Why Meilisearch over Elasticsearch?
| Criteria | Meilisearch | Elasticsearch |
|----------|-------------|--------------|
| Setup complexity | ⭐⭐⭐⭐⭐ Single binary | ⭐⭐ JVM, cluster setup |
| Resource usage | ⭐⭐⭐⭐⭐ ~256MB RAM | ⭐⭐ 2-4GB minimum |
| Typo tolerance | ⭐⭐⭐⭐⭐ Out of box | ⭐⭐⭐ Configurable |
| Speed | ⭐⭐⭐⭐⭐ <50ms queries | ⭐⭐⭐⭐ Fast |
| Cost | ⭐⭐⭐⭐⭐ Open source, low resources | ⭐⭐ Expensive at scale |
| Faceted search | ⭐⭐⭐⭐⭐ Built-in filters | ⭐⭐⭐⭐⭐ Advanced |
| For our scale | ✅ Perfect for <1M documents | Overkill |

**Decision**: Meilisearch — Lightweight, fast, excellent typo tolerance (critical for breed names), perfect for our scale, cost-effective.

---

## 7. Real-time — Socket.io

### Why Socket.io?
- **Fallback support**: WebSocket → HTTP long-polling (important for Indian networks)
- **Room-based**: Perfect for buyer-seller chat rooms
- **Scalable**: Redis adapter for multi-server support
- **NestJS integration**: First-class @nestjs/websockets support
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
| **Swagger/OpenAPI** | API documentation (auto-generated from NestJS) |

---

## 11. Development Environment

```
Required installations:
- Node.js 22.x (via nvm)
- Flutter 3.x SDK
- Docker Desktop
- PostgreSQL 16 (via Docker)
- Redis 7 (via Docker)
- Meilisearch (via Docker)
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

| Repo | Contents |
|------|----------|
| `petzonic-api` | NestJS backend |
| `petzonic-customer-app` | Flutter customer app |
| `petzonic-seller-app` | Flutter seller app |
| `petzonic-web` | Next.js website |
| `petzonic-admin` | Next.js admin panel |
| `petzonic-infra` | Terraform, Docker configs |
| `petzonic-shared` | Shared types/constants (npm package) |

**Rationale**: Simpler CI/CD, independent deployment cycles, smaller team can focus on one repo at a time. Can migrate to monorepo (Turborepo/Nx) later if needed.
