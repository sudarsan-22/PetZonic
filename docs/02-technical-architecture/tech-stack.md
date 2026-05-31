# PetZonic — Tech Stack

> **Version**: 1.1.0 (FROZEN)  
> **Date**: May 31, 2026

---

## 1. Stack Summary

| Layer | Technology | Version |
|-------|-----------|---------|
| **Mobile** | Flutter (Dart) | 3.x (latest stable) |
| **Web (Customer)** | React.js (React 19) | 15.x |
| **Web (Admin)** | React.js (React 19) | 15.x |
| **Backend** | Node.js + Express | 22.x (Node) / 5.x (Express) |
| **Language** | TypeScript | 5.x |
| **ORM** | Prisma | 6.x |
| **Database** | MongoDB + PostgreSQL | 8.x (Mongo) / 16 (PG) |
| **Cache** | Redis | 7.x |
| **Search** | Meilisearch | 1.x |
| **Queue** | BullMQ (Redis-backed) | 5.x |
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

| Purpose | Library |
|---------|---------|
| ORM | Prisma 6 |
| Validation | Zod |
| Auth | jsonwebtoken + bcryptjs |
| WebSocket | socket.io |
| Queue | BullMQ |
| Cache | ioredis |
| File Upload | multer |
| Swagger | swagger-ui-express |
| Config | dotenv |
| Throttle | express-rate-limit |
| Health Check | express-actuator / custom middleware |
| Testing | Jest + Supertest |
| Logging | pino (structured JSON logs) |
| Scheduling | node-cron |

### Backend Architecture Pattern
- **Modular Monolith** (each feature is a separate module/folder)
- **Router → Controller → Service → Repository** pattern
- **Zod schemas** for request/response validation
- **Middleware** for authentication, logging, rate limiting
- **Error handlers** for centralized exception handling
- **Feature-first** folder structure
- **Dependency injection** via factory functions

---

## 5. Database — MongoDB + PostgreSQL

### Why Both?
- **MongoDB**: Flexible schema for pets catalog, chat messages, notifications, activity logs
- **PostgreSQL**: Strict relational data for orders, payments, users, inventory, financial transactions

### Why PostgreSQL (for transactional data)?
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

### Why Prisma (ORM for PostgreSQL)?
- **Type-safe**: Auto-generated TypeScript types from schema
- **Migration system**: Version-controlled schema changes
- **Query performance**: Optimized SQL generation
- **Developer experience**: Auto-complete, intuitive API
- **Schema-first**: Single source of truth for database structure

### Why Mongoose (ODM for MongoDB)?
- **Schema validation**: Define structure for flexible documents
- **Middleware hooks**: Pre/post save, validate, remove
- **Population**: Reference-based joins across collections
- **TypeScript support**: Strong typing with interfaces
- **Mature ecosystem**: Most popular MongoDB ODM for Node.js

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
| `petzonic-api` | Node.js + Express backend |
| `petzonic-customer-app` | Flutter customer app |
| `petzonic-seller-app` | Flutter seller app |
| `petzonic-web` | React.js + Next.js website |
| `petzonic-admin` | React.js + Next.js admin panel |
| `petzonic-infra` | Terraform, Docker configs |
| `petzonic-shared` | Shared types/constants (npm package) |

**Rationale**: Simpler CI/CD, independent deployment cycles, smaller team can focus on one repo at a time. Can migrate to monorepo (Turborepo/Nx) later if needed.