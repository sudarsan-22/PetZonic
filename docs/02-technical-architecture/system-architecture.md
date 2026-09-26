# PetZonic — System Architecture

> **Version**: 1.0.0  
> **Date**: May 28, 2026 · **Accuracy-checked against source**: 2026-09-26

---

## 1. Architecture Overview

PetZonic follows a **client-server architecture** in which web clients communicate with a
single unified backend API. The backend is an **Express 5 modular monolith** in TypeScript,
using a strict **Router → Controller → Service → Repository** 4-tier architecture across
**25 domain modules** (verified 2026-09-20). It is not a microservice system.

**Built clients**: `petzonic-web` (customer, seller and provider portals in one Next.js app)
and `petzonic-admin` (separate Next.js admin console).
**Not built**: the Flutter mobile apps shown dashed in the diagram below. Their repos contain
no application code. They are included only to show intended future topology.

---

## 2. High-Level Architecture Diagram

```mermaid
graph TB
    subgraph Clients
        WEB[petzonic-web<br/>Customer + Seller + Provider<br/>Next.js 16 / React 19]
        ADM[petzonic-admin<br/>Admin Console<br/>Next.js 16 / React 19]
        CA[Customer App — NOT BUILT<br/>Flutter iOS/Android]
        SA[Seller App — NOT BUILT<br/>Flutter iOS/Android]
    end

    subgraph Load Balancer & Reverse Proxy
        NGINX[Nginx Load Balancer<br/>Failover & SSL Termination]
    end

    subgraph API Cluster
        API1[petzonic-api: Replica 1<br/>Node 22 + Express 5]
        API2[petzonic-api: Replica 2<br/>Node 22 + Express 5]
    end

    subgraph Data Layer
        PG[(PostgreSQL 16 Primary DB<br/>ACID Relational + JSONB + pg_trgm)]
        RD[(Redis 7<br/>Rate Limiting, Token Invalidation & AI Session Store)]
        S3[(AWS S3 / Cloudflare R2<br/>Object Media Storage)]
    end

    subgraph External & Local AI Services
        OLLAMA[Ollama Container<br/>Local Qwen 2.5 / Llama 3.2 LLM]
        GEMINI[Google Gemini AI<br/>Multimodal Pet Photo Analysis]
        RP[Razorpay<br/>Payments & Webhooks — mock mode if unconfigured]
        SMS[SMS Gateway<br/>Phone OTP Delivery]
    end

    WEB --> NGINX
    ADM --> NGINX
    CA -.not built.-> NGINX
    SA -.not built.-> NGINX
    NGINX --> API1
    NGINX --> API2
    API1 --> PG
    API2 --> PG
    API1 --> RD
    API2 --> RD
    API1 --> S3
    API2 --> S3
    API1 --> OLLAMA
    API2 --> OLLAMA
    API1 --> GEMINI
    API1 --> RP
    API1 --> SMS
```

> **Push notifications**: Firebase FCM is *not* wired up. The push provider in
> `petzonic-api/src/modules/notifications/providers.ts` is a permanent stub that reports
> itself unconfigured, so the notification outbox marks push sends as `SKIPPED` rather than
> pretending they were delivered.
>
> **Nginx & multi-replica**: the compose topology for this exists in `petzonic-infra`, but it
> has never been deployed. There is no TLS termination configured anywhere today.

---

## 3. Architecture Layers

### 3.1 Client Layer

| Client | Technology | Purpose | Communication |
|--------|-----------|---------|---------------|
| Website & Admin | Next.js 16 (React 19) | Customer e-commerce, listings, chat, & admin panel | REST API (Axios) + Socket.io |
| Customer App | Flutter (Dart) | iOS + Android mobile buyer experience | REST API + Socket.io |
| Seller App | Flutter (Dart) | iOS + Android mobile seller experience | REST API + Socket.io |

### 3.2 Reverse Proxy & Load Balancer

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Reverse Proxy | Nginx | SSL termination, static gzip caching, path-based routing |
| Load Balancer | Nginx Upstream | Round-robin load balancing across replicas with health failover (`max_fails=3`) |
| WebSockets | Nginx `/socket.io/` | Sticky connection upgrade for live real-time chat |

### 3.3 Application Layer (Express 5 Modular Monolith)

The backend organizes all platform capabilities into **26 domain modules** (`petzonic-api/src/modules/`, measured 2026-09-26), each following the 4-layer design:
- **Router**: Thin route definitions, middleware chains, auth & rate-limit guards
- **Controller**: HTTP request parsing, Zod validation, status codes, response envelope
- **Service**: Domain business rules, transactions, external provider integration
- **Repository**: Prisma ORM database interactions

**Active Modules:**

| Module | Scope / Responsibility |
|--------|------------------------|
| **Auth** | Registration, login, phone OTP lockout, JWT refresh rotation, Google OAuth |
| **Users** | Profile management, KYC verification, address book, roles |
| **Pets** | Pet listings, breed taxonomy, negotiable pricing, boosts, Gemini AI assist |
| **Products** | E-commerce catalog, categories, inventory, brands, **pre-owned peer listings with admin moderation** (2026-09-17) |
| **Pharmacy** | Pet medicine catalog, prescription vault and upload, customer pet profiles, admin prescription verification, Rx gating at checkout (2026-09-20) |
| **Breeders** | District-scoped breeder profiles and district browse; verification admin-granted (2026-09-20/22) |
| **Brands** | Brand directory and brand pages |
| **Support** | Customer support tickets and admin replies |
| **Metrics** | Prometheus `/metrics`, HTTP latency, subsystem probes (2026-09-17) |
| **Cart & Orders** | Server-backed cart, checkout, multi-item orders, status lifecycle, returns, **tax invoice (JSON/HTML)**, abandoned-order expiry |
| **Payments** | Razorpay order creation, HMAC webhook verification, COD, escrow holds, payouts |
| **Chat** | Socket.io real-time WebSocket chat gateway, rooms, message history |
| **Services** | Vet, grooming, sitting, training provider listings and slot bookings |
| **Reviews** | Star ratings, text reviews, helpfulness upvoting, seller/admin replies |
| **Notifications** | In-app notifications, device tokens, transactional outbox queue |
| **Community** | Discussion forums, categories, post voting, replies, lost & found board |
| **Education** | Training courses, chapters, enrollments, vet Q&A, feeding calculator |
| **Insurance** | Partner plans, coverage comparison, policy issuance, claim filing |
| **Promotions** | Discount coupons, flat/percentage rules, checkout code validation |
| **Banners** | Homepage carousel banners, schedules, link targets |
| **Media** | S3 / Cloudflare R2 upload with local disk `/uploads` fallback |
| **Newsletter** | Email subscription capture and verification |
| **Admin** | Unified admin dashboard, metrics, user moderation, dispute resolution, audit logs |
| **AI Discovery** | Conversational concierge across products, pets, breeders, services, pre-owned, pharmacy, brands, insurance and lost-and-found; multi-tab routing via `targetTab` (2026-09-23); hybrid rule-based & Ollama/Gemini intent extraction; Redis sessions |
| **Docs** | Interactive OpenAPI 3.0 Swagger UI mounted at `/api/docs` |

### 3.4 Data Layer

| Store | Technology | Purpose |
|-------|-----------|---------|
| **Primary Database** | PostgreSQL 16 | 58 tables: transactional ACID data, user accounts, listings, orders, JSONB |
| **Search Engine** | PostgreSQL `pg_trgm` | Zero-latency full-text and fuzzy trigram matching directly in DB |
| **Cache, Limiter & Session** | Redis 7 | Distributed sliding-window rate limiting (`rate-limit-redis`), AI multi-turn session cache, and token family invalidation with memory fallback |
| **Object Storage** | AWS S3 / Cloudflare R2 | Media images & documents with automated local disk fallback |

---

## 4. Communication Patterns

### 4.1 Synchronous (REST API)

```
Client ➔ Nginx ➔ Express Thin Router ➔ Controller ➔ Service ➔ Repository ➔ PostgreSQL 16
                                                            ➔ Redis 7 (rate limiting)
                                                            ➔ S3 / Cloudflare R2 (files)
```

### 4.2 Real-time (WebSocket)

```mermaid
sequenceDiagram
    participant B as Buyer App
    participant WS as WebSocket Server
    participant RD as Redis Pub/Sub
    participant S as Seller App

    B->>WS: Connect (JWT auth)
    WS->>RD: Subscribe to user channels
    B->>WS: Send message
    WS->>RD: Publish to seller channel
    RD->>WS: Deliver to seller's connection
    WS->>S: Push message to seller
    WS->>WS: Persist message to PostgreSQL
```

### 4.3 Async (Event-driven)

```mermaid
sequenceDiagram
    participant API as API Server
    participant Q as Bull Queue (Redis)
    participant W as Worker Process
    participant EXT as External Service

    API->>Q: Dispatch job (e.g., send-notification)
    Q->>W: Process job
    W->>EXT: Call FCM/SMS/Email service
    W->>Q: Mark complete or retry on failure
```

**Queues in code (2026-09-26)**: `petzonic-email-queue`, `petzonic-broadcast-queue`, and
`petzonic-maintenance-queue` — the last one runs **scheduled jobs**: `auto-release-escrows`
(hourly) and `expire-abandoned-orders` (every 15 minutes, 30-minute unpaid TTL). Workers run
inside the API process. Push delivery (FCM) is still a stub that reports itself unconfigured.

---

## 5. Data Flow — Key Scenarios

### 5.1 Pet Purchase Flow

```mermaid
sequenceDiagram
    participant Buyer
    participant API
    participant Razorpay
    participant Seller
    participant Queue

    Buyer->>API: Initiate purchase (PET-ID)
    API->>API: Validate listing, check availability
    API->>Razorpay: Create payment order (escrow)
    Razorpay-->>Buyer: Payment page
    Buyer->>Razorpay: Complete payment
    Razorpay->>API: Payment webhook (success)
    API->>API: Create order, hold in escrow
    API->>Queue: Notify seller job
    Queue->>Seller: Push: "New order received!"
    Seller->>API: Accept order
    API->>Queue: Notify buyer job
    Note over Buyer,Seller: Meetup/delivery coordination via chat
    Buyer->>API: Confirm receipt
    API->>API: Release escrow
    API->>Razorpay: Transfer to seller (minus commission)
    API->>Queue: Notify seller: payment released
```

### 5.2 Product Order Flow

```mermaid
sequenceDiagram
    participant Buyer
    participant API
    participant Razorpay
    participant Shiprocket
    participant Queue

    Buyer->>API: Place order (cart items)
    API->>API: Validate stock, calculate total
    API->>Razorpay: Create payment
    Buyer->>Razorpay: Pay
    Razorpay->>API: Webhook: paid
    API->>API: Create order, deduct stock
    API->>Shiprocket: Create shipment
    Shiprocket-->>API: AWB number, tracking URL
    API->>Queue: Order confirmation notification
    Shiprocket->>API: Webhook: status updates
    API->>Queue: Status update notification to buyer
```

### 5.3 Database-Atomic Authentication & Session Lifecycle

```mermaid
sequenceDiagram
    participant C as Browser Client
    participant API as petzonic-api
    participant DB as PostgreSQL 16
    participant RD as Redis 7

    C->>API: POST /api/v1/auth/login {email, password}
    API->>DB: Fetch user + roles (constant-time dummy fallback on missing)
    API->>API: Verify bcrypt password hash
    API->>DB: INSERT INTO refresh_tokens (family_id, token_hash, expires_at)
    API-->>C: 200 OK + HttpOnly Cookie (refreshToken) + JSON (accessToken)

    Note over C,API: Silent Token Refresh (Concurrent Collision Resilience)
    C->>API: POST /api/v1/auth/refresh (Cookie + X-Requested-With header)
    API->>DB: Atomic Lock & Revoke: UPDATE refresh_tokens SET revoked_at=NOW() WHERE id=$1 AND revoked_at IS NULL
    alt Exactly 1 Request Wins Atomic Update
        API->>DB: INSERT INTO refresh_tokens (family_id, parent_id, token_hash)
        API-->>C: 200 OK + New HttpOnly Cookie + New Access Token
    else Concurrent Race / Stale Replay Loser
        API->>DB: Replay Detected: UPDATE refresh_tokens SET revoked_at=NOW() WHERE family_id=$fam
        API-->>C: 401 Unauthorized (TOKEN_ALREADY_ROTATED / REPLAY_DETECTED)
    end
```

### 5.4 Conversational Shopping & Product Discovery Flow

```mermaid
sequenceDiagram
    participant User as Customer (Web / Mobile)
    participant Chat as AI Chat Drawer UI
    participant API as petzonic-api (/ai-discovery)
    participant RD as Redis 7 (Session Store)
    participant LLM as Ollama / Gemini Provider
    participant DB as PostgreSQL (Prisma Catalog)

    User->>Chat: "I need healthy dog food under ₹1500"
    Chat->>API: POST /api/v1/ai-discovery/chat {message, sessionId}
    API->>RD: GET session:ai-discovery:{sessionId} (context & active filters)
    API->>API: Fast-Path Rule Evaluation (species: DOG, price: <=1500, cat: dog-food)
    alt Fast-Path Matches
        API->>DB: Query products WHERE species='DOG' AND price <= 1500 AND category='dog-food'
    else Ambiguous or Complex Conversational Turn
        API->>LLM: Prompt extraction with conversation history & available categories
        LLM-->>API: JSON Structured Intent {species, category, minPrice, maxPrice, sort}
        API->>DB: Execute Prisma product query with extracted filters
    end
    API->>RD: SET session:ai-discovery:{sessionId} (sliding TTL 30m)
    API-->>Chat: 200 OK {message: "Here are the best dog foods under ₹1500", products: [...]}
    Chat-->>User: Render conversational reply + interactive ProductCard carousel
```

---

## 6. Scalability Strategy

### Phase 1 (Launch — 5K users)
- Single API server (2 vCPU, 4GB RAM)
- Single PostgreSQL instance (db.t3.medium)
- Single Redis instance
- Good enough for initial traffic

### Phase 2 (Growth — 50K users)
- Auto-scaling API servers (2-4 instances behind ALB)
- PostgreSQL read replica for read-heavy queries
- Redis cluster for sessions + cache
- Dedicated background worker container for outbox events

### Phase 3 (Scale — 500K+ users)
- Consider splitting into microservices (Chat, Payments, Notifications)
- Database sharding by region/tenant
- Dedicated WebSocket cluster
- Event-driven architecture (EventBridge/Kafka)
- Multi-region deployment

---

## 7. Fault Tolerance & Reliability

| Component | Strategy |
|-----------|----------|
| API Server | Multi-AZ deployment, auto-scaling, health checks |
| Database | Multi-AZ RDS, automated backups, point-in-time recovery |
| Redis | ElastiCache with replica, auto-failover |
| File Storage | S3 (99.999999999% durability) |
| External APIs | Circuit breaker pattern, retry with backoff, fallback responses |
| Background Jobs | Retry with exponential backoff (3 attempts), dead letter queue |

---

## 8. Monitoring & Observability

> **Implemented (2026-09-17/18)**: Prometheus + Grafana + Alertmanager + Loki/Promtail +
> cAdvisor + Node Exporter via Docker Compose, scraping the API's `/metrics`. Sentry, CloudWatch
> and X-Ray in the table below are **planned, not used**. See
> [Infrastructure §9.0](infrastructure.md#90-implemented-stack-docker-compose-since-2026-091718).

| Layer | Tool (planned) | Purpose |
|-------|------|---------|
| Application | Sentry | Error tracking, crash reporting |
| Infrastructure | CloudWatch | CPU, memory, disk, network metrics |
| Logs | CloudWatch Logs | Centralized logging (structured JSON) |
| APM | AWS X-Ray or Sentry Performance | Request tracing, slow query detection |
| Uptime | AWS Route53 Health Checks | Endpoint monitoring, alerting |
| Alerts | CloudWatch Alarms → SNS | PagerDuty/Slack integration for on-call |

---

## 9. Security Architecture

See [Security Document](security.md) for detailed security design.

**Summary:**
- All traffic over HTTPS (TLS 1.3)
- JWT authentication (short-lived access + long-lived refresh tokens)
- Role-based access control (RBAC) at API level
- Input validation on every endpoint (class-validator)
- Rate limiting per IP and per user
- File upload scanning
- No sensitive data in logs
- Secrets managed via AWS Secrets Manager

---

## 10. Deployment Architecture

```mermaid
graph TB
    subgraph AWS Mumbai Region
        subgraph Public Subnet
            ALB2[Application Load Balancer]
            CF2[CloudFront Distribution]
        end

        subgraph Private Subnet - App
            ECS[ECS Fargate<br/>API Containers]
            WORKER[ECS Fargate<br/>Worker Containers]
        end

        subgraph Private Subnet - Data
            RDS[(RDS PostgreSQL 16<br/>Multi-AZ + pg_trgm)]
            EC[(ElastiCache Redis<br/>Cluster)]
        end

        S3B[(S3 / R2 Storage<br/>Media Files)]
    end

    CF2 --> ALB2
    ALB2 --> ECS
    ECS --> RDS
    ECS --> EC
    ECS --> S3B
    WORKER --> RDS
    WORKER --> EC
```
