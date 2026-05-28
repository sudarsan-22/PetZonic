# User Stories — Admin

> **Role**: Platform Admin / Super Admin  
> **Description**: PetZonic team members managing platform operations

---

## Dashboard & Overview

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A01 | As an admin, I want to see a real-time dashboard so I know platform health | DAU/MAU, orders today, revenue today, active listings, pending approvals, system alerts |
| US-A02 | As an admin, I want to see key metrics trends over time so I track growth | Charts: users, orders, revenue, listings — daily/weekly/monthly views |
| US-A03 | As an admin, I want to receive system alerts so I act on critical issues | Email + push for: system errors, spike in reports, payment failures, server issues |

---

## User Management

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A04 | As an admin, I want to search and view any user's profile so I can investigate issues | Search by name/email/phone, view full profile, activity history, orders, listings |
| US-A05 | As an admin, I want to verify seller KYC submissions so verified sellers can list | KYC queue: view documents, approve/reject with reason, timestamp |
| US-A06 | As an admin, I want to verify breeder applications so only legit breeders get badge | Breeder queue: license validation, facility photos review, approve/reject |
| US-A07 | As an admin, I want to suspend a user who violates policies so platform stays safe | Suspend button with reason + duration (24hr/7day/30day/permanent), auto-notify user |
| US-A08 | As an admin, I want to ban a user permanently for serious violations | Ban with reason, deactivate all listings, block re-registration (phone/email) |
| US-A09 | As an admin, I want to view user reports so I can take action on violators | Reports queue: reported user, reporter, reason, evidence, action buttons |
| US-A10 | As an admin, I want to change a user's role so I can upgrade/downgrade access | Role edit: add/remove roles (seller, breeder, franchise, service provider) |

---

## Listing Moderation

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A11 | As an admin, I want to review new pet listings before they go live | Moderation queue: listing details, photos, seller info, approve/reject buttons |
| US-A12 | As an admin, I want to reject listings that violate policies with a reason | Reject with reason dropdown: inappropriate photos, misleading info, banned species, etc. |
| US-A13 | As an admin, I want to set auto-flag rules so obvious violations are caught | Rule config: flagging keywords, image detection (nudity/violence), price anomalies |
| US-A14 | As an admin, I want to bulk-approve listings from verified breeders so I'm efficient | Auto-approve toggle for verified breeders, manual review for new/unverified sellers |
| US-A15 | As an admin, I want to remove a live listing if reported as fraudulent | "Take Down" action with reason, notify seller, listing hidden immediately |
| US-A16 | As an admin, I want to manage banned species/breeds list so illegal sales are prevented | Configurable banned list (per state if needed), auto-reject listings matching |

---

## Order & Dispute Management

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A17 | As an admin, I want to view all orders and their status so I can monitor operations | Order list: search, filter by status/date/amount, view full details |
| US-A18 | As an admin, I want to resolve disputes between buyers and sellers | Dispute queue: both parties' evidence, communication history, resolution actions |
| US-A19 | As an admin, I want to initiate refunds when disputes favor the buyer | Manual refund trigger, full/partial amount, reason logged, both parties notified |
| US-A20 | As an admin, I want to release escrowed payments when disputes favor seller | Manual escrow release, evidence documented, buyer notified |
| US-A21 | As an admin, I want to cancel orders in exceptional circumstances | Force-cancel with reason, auto-refund initiated, both parties notified |

---

## Product & Catalog Management

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A22 | As an admin, I want to add new products to the store so inventory grows | Product form: name, description, photos, price, variants, category, SKU, stock |
| US-A23 | As an admin, I want to bulk-upload products via CSV so I'm efficient | CSV template download, upload + validation, error report, preview before confirm |
| US-A24 | As an admin, I want to manage product categories so catalog is organized | CRUD categories, hierarchy (parent/child), reorder, set featured categories |
| US-A25 | As an admin, I want to manage pet breed categories so listings are standardized | Species → Breed hierarchy, add new breeds, merge duplicates, set popularity order |
| US-A26 | As an admin, I want to manage inventory stock levels so products don't oversell | Stock view: current quantity, low-stock alerts, restock log, auto-disable at 0 |
| US-A27 | As an admin, I want to create deals and promotions so we drive sales | Promotion form: discount %, products/categories, validity dates, usage limits, coupon code |

---

## Franchise Management

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A28 | As an admin, I want to review franchise applications so we onboard quality partners | Application queue: business details, location, experience, approve/reject |
| US-A29 | As an admin, I want to onboard approved franchises with initial setup | Franchise setup: login credentials, inventory allocation, area assignment, training docs |
| US-A30 | As an admin, I want to monitor franchise performance so I ensure brand standards | Per-franchise dashboard: sales, ratings, complaints, compliance score |
| US-A31 | As an admin, I want to manage franchise revenue sharing so settlements are accurate | Revenue split configuration, settlement history, dispute flags |

---

## Content & Communication

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A32 | As an admin, I want to manage homepage banners so we promote relevant content | Banner CRUD: image upload, link URL, display order, schedule start/end dates |
| US-A33 | As an admin, I want to send push notifications to users so we communicate updates | Notification composer: title, body, image, target (all/segment), schedule/send now |
| US-A34 | As an admin, I want to moderate reviews that are flagged so content quality is maintained | Flagged reviews queue: review content, reason flagged, keep/remove/edit actions |
| US-A35 | As an admin, I want to manage FAQ and help content so users find answers | FAQ CRUD: question, answer, category, order; publish/unpublish toggle |

---

## Analytics & Reports

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A36 | As an admin, I want to view revenue reports so I track business health | Revenue: daily/weekly/monthly, by source (products/commission/promotions), downloadable |
| US-A37 | As an admin, I want to see user growth analytics so I measure marketing effectiveness | New signups, retention rate, churn, acquisition channel, cohort analysis |
| US-A38 | As an admin, I want to see marketplace health metrics so I balance supply/demand | Listings: new/active/expired/sold, avg time-to-sell, by category, by city |
| US-A39 | As an admin, I want to export data for reporting so I can share with stakeholders | Export: CSV/Excel, date range selection, data type selection |
| US-A40 | As an admin, I want to see search analytics so I understand user intent | Top searches, zero-result searches, trending, search-to-conversion rate |

---

## Platform Configuration

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A41 | As an admin, I want to configure payment settings so transactions work correctly | Razorpay keys, commission %, payout schedule, minimum payout amount |
| US-A42 | As an admin, I want to configure delivery settings so logistics operate smoothly | Serviceable pincodes, delivery partner config, shipping rates, free delivery threshold |
| US-A43 | As an admin, I want to use feature flags so I can roll out features gradually | Feature flag dashboard: toggle features on/off, A/B testing, user segment targeting |
| US-A44 | As an admin, I want to manage notification templates so messages are consistent | Template editor: SMS, email, push templates with variables, preview, test send |
| US-A45 | As an admin, I want to view audit logs so I can track who did what | Audit log: admin user, action, timestamp, affected entity, before/after values |

---

## Support

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A46 | As an admin, I want to manage support tickets so users get timely help | Ticket queue: priority, status, assigned agent, SLA countdown, resolution |
| US-A47 | As an admin, I want to assign tickets to team members so workload is distributed | Assign/reassign, auto-assign rules (by category), workload view |
| US-A48 | As an admin, I want to communicate with users regarding their tickets | Reply to ticket, internal notes, status update, resolution + close |

---

## Community Moderation

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A49 | As an admin, I want to moderate forum posts so community stays safe and relevant | Reported posts queue: view content, reporter reason, approve/remove/warn actions |
| US-A50 | As an admin, I want to ban users from forums for repeated violations | Forum ban: temporary (7/30 days) or permanent, ban reason recorded, user notified |
| US-A51 | As an admin, I want to pin important posts in forum categories so they're visible | Pin/unpin posts, set expiry for pinned posts, max 3 pinned per category |
| US-A52 | As an admin, I want to manage forum categories so community is well organized | Add/edit/reorder categories, set category icons, archive inactive categories |
| US-A53 | As an admin, I want to verify lost & found posts so fake reports are prevented | Review queue for lost/found, verify with user, mark resolved, prevent abuse |

---

## Educational Content Management

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A54 | As an admin, I want to review submitted content before publishing so quality is maintained | Content review queue: preview, approve/reject/request changes, publish date |
| US-A55 | As an admin, I want to manage educational categories and tags so content is discoverable | Category CRUD, tag management, featured content selection |
| US-A56 | As an admin, I want to manage video courses so platform offers quality learning | Course review: check lessons, approve/reject, feature on homepage |
| US-A57 | As an admin, I want to manage vet profiles for telemedicine so only qualified vets consult | Vet telemedicine approval: verify license, specialization, set consultation limits |
| US-A58 | As an admin, I want to view content analytics so I know what users want to learn | Content metrics: views, likes, completion rates, top content, engagement trends |

---

## Insurance Partner Management

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-A59 | As an admin, I want to onboard insurance partners so we offer multiple plan options | Partner setup: company details, API credentials, commission %, plan upload |
| US-A60 | As an admin, I want to manage insurance plans offered on the platform | Plan CRUD: coverage details, pricing, enable/disable plans, set display order |
| US-A61 | As an admin, I want to track insurance sales and commissions so I monitor revenue | Insurance dashboard: policies sold, premiums collected, commission earned, by partner |
| US-A62 | As an admin, I want to oversee claim disputes between users and insurance partners | Claim escalation queue: user complaint, partner response, mediate, resolve |
| US-A63 | As an admin, I want to view insurance analytics so I optimize plan offerings | Analytics: popular plans, conversion rate, claim ratio, partner performance |
