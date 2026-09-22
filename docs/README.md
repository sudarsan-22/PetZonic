# PetZonic — Project Documentation

> **Status**: Pre-launch — feature-complete in development, not yet deployed to production  
> **Last Updated**: 2026-09-20  
> **Version**: 1.2.0
>
> **Verified against source on 2026-09-20.** Counts in this index are measured from the
> working tree, not estimated. See [Current State](#current-state) for the authoritative numbers.

---

## Overview

PetZonic is the world's first multi-platform pet ecosystem combining:
- **E-commerce store** (accessories, pet products — PetZonic-owned inventory)
- **C2C/B2C Marketplace** (pet buying/selling by breeders, brokers, individuals)
- **Services platform** (veterinary, pet care, grooming)
- **Franchise network** (brand outlets under PetZonic)

**Platforms (built)**: Website (customer + seller + provider portals), Admin Panel  
**Platforms (planned, not started)**: iOS App, Android App — the `petzonic-customer-app` and
`petzonic-seller-app` repos are empty stubs containing no application code  
**Target Market**: India (initially)

---

## Documentation Index

### 01 — Product Requirements
| Document | Description |
|----------|-------------|
| [PRD](01-product-requirements/PRD.md) | Master product requirements document |
| [Feature List](01-product-requirements/feature-list.md) | Complete frozen feature list with acceptance criteria |
| [Competitor Analysis](01-product-requirements/competitor-analysis.md) | Indian pet market landscape |
| **User Stories** | |
| [Buyer Stories](01-product-requirements/user-stories/buyer-stories.md) | End-user/customer journeys |
| [Seller Stories](01-product-requirements/user-stories/seller-stories.md) | Individual seller & broker journeys |
| [Breeder Stories](01-product-requirements/user-stories/breeder-stories.md) | Verified breeder journeys |
| [Admin Stories](01-product-requirements/user-stories/admin-stories.md) | Platform administration |
| [Service Provider Stories](01-product-requirements/user-stories/service-provider-stories.md) | Vet & pet care provider journeys |

### 02 — Technical Architecture
| Document | Description |
|----------|-------------|
| [System Architecture](02-technical-architecture/system-architecture.md) | High-level system design & diagrams |
| [Tech Stack](02-technical-architecture/tech-stack.md) | Technology choices & justification |
| [Infrastructure](02-technical-architecture/infrastructure.md) | AWS deployment & DevOps |
| [Security](02-technical-architecture/security.md) | Authentication, authorization, data protection |
| [Third-Party Integrations](02-technical-architecture/third-party-integrations.md) | External services & APIs |

### 03 — Database Design
| Document | Description |
|----------|-------------|
| [ER Diagram](03-database-design/er-diagram.md) | Entity-relationship diagram (Mermaid) |
| [Schema](03-database-design/schema.md) | All tables, columns, types, indexes |
| [Data Dictionary](03-database-design/data-dictionary.md) | Field definitions, constraints, business rules |

### 04 — API Design
| Document | Description |
|----------|-------------|
| [API Overview](04-api-design/api-overview.md) | Conventions, versioning, authentication |
| [Auth API](04-api-design/auth-api.md) | Registration, login, OTP, tokens |
| [Pets API](04-api-design/pets-api.md) | Pet listings CRUD, search, filters |
| [Products API](04-api-design/products-api.md) | Store products & inventory |
| [Orders API](04-api-design/orders-api.md) | Cart, checkout, order lifecycle |
| [Payments API](04-api-design/payments-api.md) | Payment processing & refunds |
| [Chat API](04-api-design/chat-api.md) | Messaging & WebSocket events |
| [Services API](04-api-design/services-api.md) | Vet & pet care bookings |
| [Pharmacy API](04-api-design/pharmacy-api.md) | 🆕 Medicine catalog, prescriptions, admin verification |
| [Breeders API](04-api-design/breeders-api.md) | 🆕 District-scoped breeder profiles (backend only, no UI yet) |
| [Reviews API](04-api-design/reviews-api.md) | Ratings & reviews |
| [Admin API](04-api-design/admin-api.md) | Administration endpoints |
| [Notifications API](04-api-design/notifications-api.md) | Push, in-app, SMS triggers |

### 05 — UI/UX Design
| Document | Description |
|----------|-------------|
| [User Flows](05-ui-ux/user-flows.md) | Key journey diagrams |
| [Screen Inventory](05-ui-ux/screen-inventory.md) | Complete screen list per app |
| [Customer App Screens](05-ui-ux/customer-app-screens.md) | Mobile app screen descriptions |
| [Seller App Screens](05-ui-ux/seller-app-screens.md) | Seller app screen descriptions |
| [Website Pages](05-ui-ux/website-pages.md) | Web page descriptions |
| [Admin Panel Pages](05-ui-ux/admin-panel-pages.md) | Admin dashboard screens |

### 06 — Project Roadmap
| Document | Description |
|----------|-------------|
| [Milestones](06-project-roadmap/milestones.md) | Development phases & deliverables |
| [Dependencies](06-project-roadmap/dependencies.md) | Task dependencies & critical path |
| [Team Roles](06-project-roadmap/team-roles.md) | Required skills & responsibilities |

### 07 — Development Guide
| Document | Description |
|----------|-------------|
| [Coding Standards](07-development-guide/coding-standards.md) | Naming, formatting, TypeScript/Flutter/Next.js conventions |
| [Git Workflow](07-development-guide/git-workflow.md) | Branching strategy, PRs, commit conventions |
| [Local Setup](07-development-guide/local-setup.md) | Step-by-step dev environment setup |
| [Testing Strategy](07-development-guide/testing-strategy.md) | Unit, integration, E2E testing approach |
| [Deployment Runbook](07-development-guide/deployment-runbook.md) | Production deployment, rollback, incident response |

### 08 — Application Visual Diagrams
| Document | Description |
|----------|-------------|
| [Application Interconnection](08-appication-visual%20diagram/application-interconnection.md) | Mermaid diagrams: role relationships, nav/dashboard separation, core marketplace flow, actual system architecture, admin scope |

### 09 — Audit Reports
| Document | Description |
|----------|-------------|
| [Database & Redis Security Audit](09-audit-reports/database-redis-security-audit.md) | DB/Redis security, privacy, and data-integrity audit (2026-09-10). Carries an accuracy notice — several claims (model inventory, password hashing) don't match the codebase; see the notice at the top of the file. |
| [End-to-End Audit Report](09-audit-reports/end-to-end-audit-report.md) | Application-wide functional + security audit (2026-09-10). Mostly consistent with the codebase; see accuracy notice for the specific exceptions. |
| [Final Production Hardening Report](09-audit-reports/final-production-hardening-report.md) | Claimed production-hardening changes (2026-09-10). Least reliable of the three — claims a migration and CHECK constraints that don't exist in the repo; see accuracy notice. |

---

## Quick Reference

- **Tech Stack**: Next.js 16 (React 19) web & admin · Node.js 22 + Express 5 (TypeScript) backend · PostgreSQL 16 (Prisma 7, 68 models) · Redis 7 · BullMQ · Socket.IO 4 · AWS S3 / local disk fallback
- **Apps (built)**: `petzonic-web` (customer + seller + provider portals) · `petzonic-admin` (admin console) · `petzonic-api` (REST + WebSocket backend) · `petzonic-infra` (Docker, Terraform, monitoring)
- **Apps (stubs, no code)**: `petzonic-customer-app`, `petzonic-seller-app`
- **User Roles**: the `Role` enum has exactly four values — `BUYER`, `SELLER`, `BREEDER`, `ADMIN`. Vet and pet-caretaker are modelled as `ServiceProvider` records, not roles. Broker and franchise are not implemented.
- **Payments**: Razorpay (UPI, Cards, Wallets, escrow hold/release, COD) — runs in mock mode when unconfigured
- **Target**: India (English; Hindi not yet implemented)

---

## Current State

Measured directly from the working tree on **2026-09-20**:

| Metric | Actual |
|---|---|
| Backend modules (`petzonic-api/src/modules/`) | 26 |
| Prisma models | 68 |
| Prisma enums | 47 |
| Migrations | 28 |
| Routers mounted under `/api/v1` | 36 |
| Web pages (`petzonic-web`) | 86 |
| Admin pages (`petzonic-admin`) | 29 |
| Backend test files | 46 |
| Web test files | 99 (+ 7 Playwright e2e specs) |
| Admin test files | 0 |

**Deployment status**: never deployed. Terraform has not been applied, no TLS is
configured, and no Razorpay/AWS production accounts are set up (see
[dependencies](06-project-roadmap/dependencies.md)).

**Most recent features** (added 2026-09-19/20):
- **Pharmacy** — fully wired end to end: `PharmacyProduct`, `Prescription`, `PrescriptionItem`,
  `OrderPrescription`, `PharmacySellerProfile`, `UserPetProfile` models; `/api/v1/pharmacy` and
  `/api/v1/admin/pharmacy` routes; customer pages at `/pharmacy` and `/pharmacy/products/[slug]`.
- **Breeders & district discovery** — full stack: `BreederProfile` model, a `district` column on
  `pet_listings`, the `/api/v1/breeders` routes, and a `/breeders` directory page with
  district browse and per-district counts. This is the feature that lets buyers find breeders
  directly instead of defaulting to shops.
