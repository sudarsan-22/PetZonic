# Pet Registry, Verified Network & Pet Data Insights — Feature Design

> **Status**: Proposed — design approved for documentation, **not implemented**  
> **Last Updated**: 2026-09-26  
> **Owner**: Founding team  
> **Repos affected**: `petzonic-api`, `petzonic-web`, `petzonic-admin`

---

## 1. Summary

PetZonic will become India's de-facto pet registry by giving every pet on the platform a
lifelong **PetZonic Pet ID**, tied to a permanent physical mark on the animal (bird leg ring,
microchip). On top of that registry it adds two products:

1. **PetZonic Verified** — a badge for pet shops, breeders, pharmacies and vets that passed a
   document and premises check, discoverable through a **Verified near me** search.
2. **Pet data insights and brand promotion** — aggregated, consented population data (species,
   breed, age, district) that brands pay to reach and to read, without ever receiving an
   individual owner's personal data.

## 2. Problem

- **No one knows how many pets India has.** There is no reliable national count of pets by
  breed, city or owner type.
- **Government registration misses most pets.** Municipal licences and government portals cover
  a small share of owners. Home and backyard breeders, who produce litters every season, almost
  never register there.
- **KCI covers pedigree dogs only.** Kennel Club of India registration tracks a paying minority
  of pure-bred dogs. Cats, birds, mixed breeds and unregistered litters are invisible.
- **Breeders post where buyers are.** A home breeder will not file a government form but will
  list a litter on PetZonic to sell it. Every listing, pet profile and sale is a data point.
- **Buyers cannot tell good sellers from bad.** Offline shops and breeders have no trust signal.

## 3. Goals and non-goals

**Goals**

1. Every pet on PetZonic has one persistent Pet ID, whether it came from a listing, a purchase
   or an owner's own profile.
2. Track lineage and litters (sire, dam, litter, breeder) — a KCI-style record for pets KCI
   never sees.
3. Tie each Pet ID to a permanent physical mark so the record proves it is the right animal.
4. Manage the pet's whole life on its Pet ID — vaccinations, deworming, vet visits, sale,
   insurance, death — with due-date reminders (the health passport).
5. Publish aggregated pet population insights by species, breed, district and age, refreshed
   at least weekly.
6. Let brands promote to relevant owner segments without receiving personal data.
7. Launch PetZonic Verified with a real admin review workflow and a "near me" search.

**Non-goals (this phase)**

- Replacing KCI or municipal licensing, or claiming legal registration status.
- Selling or exporting raw, row-level user data to any third party.
- Manufacturing microchips or rings (PetZonic partners with suppliers).
- Mobile apps (`petzonic-customer-app` / `petzonic-seller-app` are still empty stubs).

## 4. Decisions (2026-09-26)

| Topic | Decision |
|---|---|
| Launch area | All of India from day one, not a single city |
| Premises check | Both: video call for every applicant; field visit also for pet shops, breeders with 5+ breeding animals, and any business with an upheld complaint |
| Pet ID format | Includes the state code of **first registration**, e.g. `PZ-TN-2026-000123`; never changes when the pet moves |
| Pet shops | Sell through PetZonic checkout **and** are listed in `/near-me` |
| Pet page visibility | Public by default in a limited form (breed, age, lineage, breeder, trust level, district). Owner name, contact and address never public. Owner can switch to private |
| Rollout order | Verified network first, then registry, then health passport, then insights and brands |
| Extra features | Lost/stolen alert + QR tag, easy breeder onboarding + score, health guarantee, city licence helper, adoption & shelters — all added (2026-09-26) |

## 5. Document map

| # | Document | Contents |
|---|---|---|
| 01 | [Pet Registry & Lineage](01-pet-registry.md) | Pet ID, how pets enter the registry, lineage, ownership transfer |
| 02 | [Pet Identity Marks](02-pet-identity-marks.md) | Leg rings, microchips, nose prints, trust levels, handover check |
| 03 | [Verified Network & Near Me](03-verified-network.md) | Verification workflow, checklists, near-me search, business model |
| 04 | [Data Insights & Brand Promotion](04-data-insights-and-brands.md) | Insights, brand products, suppression rules |
| 05 | [Data Model & API](05-data-model-and-api.md) | Prisma models, endpoints, jobs, per-repo changes |
| 06 | [UI Design](06-ui-design.md) | Badge system, screens, wireframes, matched to current `petzonic-web` style |
| 07 | [Privacy & DPDP Compliance](07-privacy-and-compliance.md) | Consent, purpose limitation, retention |
| 08 | [Suppliers & Pricing](08-suppliers-and-pricing.md) | Indian ring/microchip/vet partners, benchmarks, proposed prices |
| 09 | [Rollout Plan](09-rollout-plan.md) | Phases, gates, metrics, open items |
| 10 | [Pet Lifecycle & Health Passport](10-pet-lifecycle.md) | Birth-to-death timeline, vaccinations, schedules, reminders, certificate |
| 11 | [Lost & Stolen Alert, QR Tag](11-lost-stolen-and-qr-tag.md) | Lost/stolen status on Pet ID, masked owner contact, stolen-listing block, QR collar tag |
| 12 | [Breeder Onboarding & Score](12-breeder-onboarding-and-score.md) | 3-step litter registration via WhatsApp link, free rings, referrals, breeder score |
| 13 | [Health Guarantee at Sale](13-health-guarantee-at-sale.md) | 7-day vet-checked guarantee on escrow, health disputes, insurance at handover |
| 14 | [City Licence Helper](14-city-licence-helper.md) | Licence pack from Pet ID, city rules, renewals, corporation partnerships |
| 15 | [Adoption & Shelters](15-adoption-and-shelters.md) | Verified shelters, free adoption transfers, community animals |
| 16 | [Future Ideas](16-future-ideas.md) | DNA tests, nose-print matching, birthday offers, and more |

## 6. How it fits the current application

All three features extend existing modules and models; no new service or repo is needed.
Current state measured 2026-09-26: 26 API modules, 68 Prisma models.

| Need | What already exists | Gap to close |
|---|---|---|
| Owner's own pets | `UserPetProfile` (name, species, breed, DOB, weight, allergies), used by pharmacy | No public Pet ID, no location, no lineage; species/breed are free text |
| Pets for sale | `PetListing` with `speciesId`, `breedId`, `city`, `district`, `state`, `ageMonths` | Not linked to a pet record; history ends at sale |
| Breeders | `BreederProfile` with district/state/specialization; `/api/v1/breeders` | `isVerified` became admin-granted (default `false`) in migration `20260922010000_breeder_verification_admin_granted`, but no admin endpoint or screen grants it yet |
| Shops & providers | `ServiceProvider` with `PENDING/APPROVED/REJECTED`, `isVerified`, lat/lng, haversine near-me search (0.1–50 km) | No retail pet-shop type |
| Seller trust | `KycSubmission` (`SELLER`/`BREEDER`/`PROVIDER`) reviewed at admin `/kyc` | KYC approval is not shown to buyers |
| Brand promotion | `Banner`, `Coupon`, `ProductBrand`, admin broadcast | No audience targeting, no brand reporting |
| Consent | `NotificationPreference` (channels only) | No data-use or marketing consent anywhere |
| Scheduled jobs | `petzonic-maintenance-queue` (since 2026-09-22): escrow auto-release hourly, abandoned-order expiry every 15 min | Add new tasks: snapshots, verification expiry, reminders, breeder score, licence renewals, adoption follow-ups |

The design follows existing house rules: 4-tier modules (router → controller → service →
repository), separate `admin-<domain>.router.ts` files, `authorize(...)` on every privileged
API route, Zod at the edge, and "degrade gracefully, never fake success".
