# PetZonic — Application Interconnection Diagrams

> How the Customer, Seller, Breeder, Service Provider, and Admin surfaces of
> `petzonic-web` actually connect to each other and to the backend, as
> currently implemented (not the original aspirational spec — see
> `02-technical-architecture/system-architecture.md` for that).

---

## 1. User Roles at a Glance

| Role | Tracked via | Primary surface |
|---|---|---|
| **Buyer** | `Role.BUYER` (default on signup) | Customer site (`/pets`, `/products`, `/services`, `/cart`, `/checkout`, `/account`) |
| **Seller** | `Role.SELLER` (granted after KYC approval) | Seller Dashboard (`/seller/*`) |
| **Breeder** | `Role.BREEDER` (granted after KYC + license approval) | Seller Dashboard (`/seller/*`, same as Seller) |
| **Service Provider** (Vet/Groomer/Sitter/Walker/Trainer) | Separate `ServiceProvider` row + status (`PENDING`/`APPROVED`/`REJECTED`) — **not** a `Role` | Provider Dashboard (`/provider/*`) |
| **Admin** | `Role.ADMIN` | Admin Panel — separate `petzonic-admin` app, routes at its root (`/users`, `/kyc`, …) |

A single logged-in user can hold multiple roles at once (e.g. a Buyer who is
also a Seller) — the UI below reflects that everywhere it says "role-aware."

---

## 2. Role Interconnection Map

```mermaid
graph LR
    Buyer["👤 Buyer"]
    Seller["🏪 Seller / Breeder"]
    Provider["🩺 Service Provider"]
    Admin["🛡️ Admin"]

    Buyer -- "browse listings" --> Seller
    Buyer -- "chat / negotiate" --> Seller
    Buyer -- "place order + payment" --> Seller
    Buyer -- "review after purchase" --> Seller

    Buyer -- "browse services" --> Provider
    Buyer -- "book / vet consultation" --> Provider
    Buyer -- "review after service" --> Provider

    Buyer -- "post / reply" --> Community(("Community<br/>Lost &amp; Found"))
    Seller -- "post / reply" --> Community
    Provider -- "post / reply" --> Community

    Seller -- "submit KYC + license" --> Admin
    Provider -- "submit registration" --> Admin
    Buyer -- "submit KYC (to become Seller)" --> Admin

    Admin -- "approve / reject" --> Seller
    Admin -- "approve / reject" --> Provider
    Admin -- "moderate posts / reviews / listings" --> Community
    Admin -- "resolve disputes between" --> Buyer
    Admin -- "resolve disputes between" --> Seller
    Admin -- "release payouts to" --> Seller
    Admin -- "release payouts to" --> Provider

    Notif{{"🔔 Notifications<br/>(cross-cutting)"}}
    Buyer -.-> Notif
    Seller -.-> Notif
    Provider -.-> Notif
    Admin -.-> Notif
```

---

## 3. Navigation & Interface Separation

Every page used to share one customer `Navbar` (shopping links + Cart +
Wishlist) no matter which section you were in. Fixed so business-management
sections get a minimal, purpose-built header instead — same separation
Amazon Seller Central / OLX's Dealer App use.

```mermaid
graph TB
    subgraph Customer-Facing["Customer-facing chrome — Navbar.tsx"]
        NAV["Navbar<br/>Pets · Products · Services · Community · Learn · Insurance<br/>Search · Wishlist · Cart · Chat · Notifications"]
        NAV --> RoleLinks{"Role-aware links"}
        RoleLinks -- "SELLER / BREEDER role" --> SellBtn["'Seller Dashboard' button"]
        RoleLinks -- "no seller role" --> SellBtn2["'Sell' button → /sell"]
        RoleLinks -- "ADMIN role" --> AdminLink["'Admin' link"]
    end

    subgraph Business-Facing["Business-management chrome — DashboardHeader.tsx"]
        DH["DashboardHeader<br/>Logo · Section label · ← Marketplace · Notifications · Logout"]
        DH --> AdminHeader["AdminHeader → /admin/*<br/>(no chat)"]
        DH --> SellerHeader["SellerHeader → /seller/*<br/>(+ Chat)"]
        DH --> ProviderHeader["ProviderHeader → /provider/*<br/>(+ Chat)"]
    end

    SellBtn -- "enters" --> SellerHeader
    AdminLink -- "enters" --> AdminHeader
    AccountPage["/account quick-links<br/>(Seller / Admin / Provider Dashboard cards)"] -- "enters" --> AdminHeader
    AccountPage -- "enters" --> SellerHeader
    AccountPage -- "enters" --> ProviderHeader
    AdminHeader -- "← Marketplace" --> NAV
    SellerHeader -- "← Marketplace" --> NAV
    ProviderHeader -- "← Marketplace" --> NAV
```

---

## 4. Core Marketplace Flow (Buy a Pet, end to end)

```mermaid
sequenceDiagram
    actor B as Buyer
    participant Web as petzonic-web
    participant API as petzonic-api
    actor S as Seller
    actor A as Admin

    S->>Web: Create pet listing (+photos)
    Web->>API: POST /pets
    API-->>A: Listing queued for moderation
    A->>API: Approve listing
    API-->>Web: Listing goes live

    B->>Web: Browse / search pets
    Web->>API: GET /pets?filters
    API-->>B: Listing results

    B->>Web: Open listing, start chat
    Web->>API: POST /chat/conversations
    B-->>S: Negotiate price / ask questions

    B->>Web: Place order + pay
    Web->>API: POST /orders, POST /payments
    API-->>API: Payment captured, held in escrow

    S->>Web: Mark shipped + tracking number
    B->>Web: Confirm receipt
    Web->>API: PATCH /orders/:id (status=DELIVERED)
    API-->>API: Escrow released to Seller balance

    B->>Web: Leave a review
    S->>Web: View payout in Seller Dashboard
    API-->>S: Payout available (admin-processed batch)
```

---

## 5. Actual System Architecture

Updated 2026-09-26 to match the code (the previous diagram said "no Redis yet" and showed
only the web app).

```mermaid
graph TB
    subgraph Clients
        WEB["petzonic-web<br/>Next.js 16 — customer, seller, provider<br/>rewrites /api/v1/* to the API"]
        ADM["petzonic-admin<br/>Next.js 16 — separate app, root routes"]
    end

    subgraph Backend["petzonic-api (Express 5 modular monolith, 26 modules)"]
        EXP["REST /api/v1 — 36 routers"]
        AUTH["JWT access + rotating refresh tokens<br/>authorize() re-checks DB"]
        SOCKET["Socket.IO /chat and /consultation"]
        AI["AI concierge<br/>multi-tab routing"]
        JOBS["BullMQ workers<br/>email · broadcast · maintenance"]
    end

    subgraph Data
        PG[("PostgreSQL 16<br/>Prisma 7, 68 models")]
        REDIS[("Redis 7<br/>queues, rate limits, AI sessions")]
        FILES[("S3 or local /uploads")]
    end

    subgraph External["External services (degrade gracefully if unset)"]
        RZP["Razorpay"]
        GOOG["Google OAuth"]
        LLM["Ollama / Gemini"]
        MSG["SMTP · MSG91/Twilio"]
        SHIP["Shiprocket / Delhivery"]
    end

    subgraph Ops["petzonic-infra monitoring"]
        PROM["Prometheus · Grafana<br/>Alertmanager · Loki"]
    end

    WEB -- "axios /api/v1/*" --> EXP
    ADM -- "axios /api/v1/admin/*" --> EXP
    WEB -- "socket.io" --> SOCKET
    EXP --> AUTH
    EXP --> AI
    EXP --> PG
    EXP --> REDIS
    EXP --> FILES
    JOBS --> REDIS
    JOBS --> PG
    EXP -.-> RZP
    EXP -.-> GOOG
    AI -.-> LLM
    JOBS -.-> MSG
    EXP -.-> SHIP
    PROM -- "scrape /metrics" --> EXP
```

The maintenance queue runs scheduled jobs: escrow auto-release (hourly) and abandoned-order
expiry (every 15 minutes).

---

## 6. What Admin Can Touch

```mermaid
graph LR
    Admin["🛡️ petzonic-admin (separate app)"]

    Admin --> Users["Users<br/>(search, suspend, roles)"]
    Admin --> KYC["KYC queue<br/>(approve/reject sellers)"]
    Admin --> Providers["Provider approvals"]
    Admin --> Listings["Listings / Products / Categories<br/>(moderation)"]
    Admin --> Community["Community posts<br/>(pin, remove)"]
    Admin --> Reviews["Reviews<br/>(respond, remove)"]
    Admin --> Disputes["Disputes<br/>(buyer vs seller)"]
    Admin --> Orders["Orders<br/>(browse all)"]
    Admin --> Payouts["Payouts &amp; Revenue"]
    Admin --> Rx["Pharmacy prescriptions<br/>(verify, 2026-09-20)"]
    Admin --> PreOwned["Pre-owned gear<br/>(approve/reject, 2026-09-17)"]
    Admin --> Promotions["Coupons / Promotions"]
    Admin --> Banners["Homepage Banners"]
    Admin --> Insurance["Insurance partners &amp; plans"]
    Admin --> Education["Education courses"]
    Admin --> Notify["Send Notification<br/>(broadcast)"]
    Admin --> Audit["Audit Log"]
    Admin --> Settings["Platform Settings"]
```

---

## 7. Basic UI Wireframes

Simple text wireframes of each shell as actually implemented — the header
row is what differs between them (see §3); page content below the header
is representative, not exhaustive.

### 7.1 Customer site — `Navbar.tsx` (shopping chrome)

```
+--------------------------------------------------------------------+
| 🐾 PetZonic   Pets  Products  Services  Community  Learn  Insurance |
|                        🔍  ❤(2)  🛒(3)  🔔  💬   Hi, Rahul  Logout  |
|                                                    [ Sell ]         |  <- "Seller Dashboard"
+--------------------------------------------------------------------+  instead, if user has
|                                                                      |  SELLER/BREEDER role
|   [ Hero banner ]                                                   |
|                                                                      |
|   Featured Pets                                                     |
|   +--------+  +--------+  +--------+  +--------+                   |
|   | photo  |  | photo  |  | photo  |  | photo  |                    |
|   | name   |  | name   |  | name   |  | name   |                    |
|   | price  |  | price  |  | price  |  | price  |                    |
|   +--------+  +--------+  +--------+  +--------+                   |
|                                                                      |
+--------------------------------------------------------------------+
| About Us · Careers · Blog   Help · Contact · Track Order            |
| Terms · Privacy · Refund     [ Newsletter signup form ]             |
+--------------------------------------------------------------------+
```

### 7.2 Seller Dashboard — `SellerHeader.tsx`

```
+--------------------------------------------------------------------+
| 🐾 PetZonic | Seller Dashboard        ← Marketplace 🔔 💬 Hi,Asha Logout |
+--------------------------------------------------------------------+
|                                                                      |
|  Seller Dashboard                                                   |
|  Total Listings: 12   Active: 9   Pending Orders: 3   Earnings: ₹.. |
|                                                                      |
|  +----------------+  +----------------+  +----------------+        |
|  | 📦 My Listings  |  | 🛍️ Orders       |  | 💰 Payouts &    |        |
|  | Manage pet      |  | Incoming orders |  |    Earnings     |        |
|  | listings     →  |  | for listings →  |  | Balance/hist →  |        |
|  +----------------+  +----------------+  +----------------+        |
|                                                                      |
+--------------------------------------------------------------------+
    (no marketing footer — dashboard ends here)
```

### 7.3 Admin Panel — `AdminHeader.tsx`

```
+--------------------------------------------------------------------+
| 🐾 PetZonic | Admin Panel              ← Marketplace 🔔  Logout      |
+--------------------------------------------------------------------+
|                                                                      |
|  Admin Dashboard        [Today] [Week] [Month] [Year]  Moderation → |
|                                                                      |
|  +-------------+  +-------------+  +-------------+  +-------------+ |
|  | DAU/Orders  |  | Revenue     |  | Active       |  | Pending     | |
|  | today       |  | today       |  | listings     |  | approvals   | |
|  +-------------+  +-------------+  +-------------+  +-------------+ |
|                                                                      |
|  Quick links: Users · KYC · Disputes · Payouts · Providers ·        |
|               Listings · Community · Reviews · Insurance ·          |
|               Promotions · Banners · Education · Audit Log          |
|                                                                      |
+--------------------------------------------------------------------+
```

### 7.4 Provider Dashboard — `ProviderHeader.tsx`

```
+--------------------------------------------------------------------+
| 🐾 PetZonic | Provider Dashboard      ← Marketplace 🔔 💬 Hi,Dr.P Logout |
+--------------------------------------------------------------------+
|                                                                      |
|  🩺 Service Provider Dashboard                                       |
|  Dr. Priya's Clinic · Veterinary · Bangalore, KA      [ APPROVED ]  |
|                                                                      |
|  +----------------+  +----------------+  +----------------+        |
|  | 📋 Services     |  | 📅 Schedule     |  | 🗂️ Bookings     |        |
|  | Manage bookable |  | Weekly          |  | Incoming        |        |
|  | services     →  |  | availability →  |  | customer     →  |        |
|  +----------------+  +----------------+  +----------------+        |
|  +----------------+   (Consultations card only for VET_CONSULTATION) |
|  | 🩺 Consultations|                                                  |
|  | Complete booked |                                                  |
|  | vet consults  → |                                                  |
|  +----------------+                                                  |
|                                                                      |
+--------------------------------------------------------------------+
```
