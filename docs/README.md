# PetZonic — Project Documentation

> **Status**: Feature Freeze | Documentation Phase  
> **Last Updated**: May 28, 2026  
> **Version**: 1.0.0

---

## Overview

PetZonic is a multi-platform pet ecosystem combining:
- **E-commerce store** (accessories, pet products — PetZonic-owned inventory)
- **C2C/B2C Marketplace** (pet buying/selling by breeders, brokers, individuals)
- **Services platform** (veterinary, pet care, grooming)
- **Franchise network** (brand outlets under PetZonic)

**Platforms**: iOS App, Android App, Website, Admin Panel  
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

---

## Quick Reference

- **Tech Stack**: Flutter (mobile) · Next.js (web) · NestJS (backend) · PostgreSQL · Redis · AWS
- **Apps**: Customer App · Seller App · Website · Admin Panel
- **User Roles**: Buyer · Seller · Breeder · Broker · Franchise · Vet · Pet Caretaker · Admin
- **Payments**: Razorpay (UPI, Cards, Wallets)
- **Target**: India (Hindi + English)
