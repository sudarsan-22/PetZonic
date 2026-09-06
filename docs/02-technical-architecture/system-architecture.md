# PetZonic — System Architecture

> **Version**: 1.0.0  
> **Date**: May 28, 2026

---

## 1. Architecture Overview

PetZonic follows a **client-server architecture** with web and mobile frontend clients communicating with a single unified backend API. The backend is structured as an **Express 5 Modular Monolith** in TypeScript, adopting a strict **Router → Controller → Service → Repository** 4-tier architecture across 19 domain modules.

---

## 2. High-Level Architecture Diagram

```mermaid
graph TB
    subgraph Clients
        WEB[Website & Admin Portal<br/>Next.js 16 / React 19]
        CA[Customer App<br/>Flutter iOS/Android]
        SA[Seller App<br/>Flutter iOS/Android]
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
        RD[(Redis 7<br/>Shared Rate Limiter & Cache)]
        S3[(AWS S3 / Cloudflare R2<br/>Object Media Storage)]
    end

    subgraph External Services
        RP[Razorpay<br/>Payments & Webhooks]
        GEMINI[Google Gemini AI<br/>Pet Photo Analysis]
        FCM[Firebase FCM<br/>Push Notifications]
        SMS[SMS Gateway<br/>Phone OTP Delivery]
    end

    WEB --> NGINX
    CA --> NGINX
    SA --> NGINX
    NGINX --> API1
    NGINX --> API2
    API1 --> PG
    API2 --> PG
    API1 --> RD
    API2 --> RD
    API1 --> S3
    API2 --> S3
    API1 --> RP
    API1 --> GEMINI
    API1 --> FCM
    API1 --> SMS
```

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

The backend organizes all platform capabilities into 19 cohesive domain modules, each following the 4-layer design:
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
| **Products** | E-commerce catalog, categories, variants, inventory management |
| **Cart & Orders** | Shopping cart, checkout, multi-item orders, order status lifecycle, returns |
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
| **Docs** | Interactive OpenAPI 3.0 Swagger UI mounted at `/api/docs` |

### 3.4 Data Layer

| Store | Technology | Purpose |
|-------|-----------|---------|
| **Primary Database** | PostgreSQL 16 | 58 tables: transactional ACID data, user accounts, listings, orders, JSONB |
| **Search Engine** | PostgreSQL `pg_trgm` | Zero-latency full-text and fuzzy trigram matching directly in DB |
| **Cache & Limiter** | Redis 7 | Distributed sliding-window rate limiting (`rate-limit-redis`) with memory fallback |
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
- Meilisearch dedicated instance

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

| Layer | Tool | Purpose |
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
            RDS[(RDS PostgreSQL<br/>Multi-AZ)]
            EC[(ElastiCache Redis<br/>Cluster)]
            MS2[(Meilisearch<br/>EC2)]
        end

        S3B[(S3 Bucket<br/>Media Files)]
    end

    CF2 --> ALB2
    ALB2 --> ECS
    ECS --> RDS
    ECS --> EC
    ECS --> MS2
    ECS --> S3B
    WORKER --> RDS
    WORKER --> EC
```
