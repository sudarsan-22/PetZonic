# 05 — Data Model, API & Per-Repo Changes

> **Status**: Proposed — not implemented · **Last Updated**: 2026-09-26  
> Back to [feature overview](README.md)

The work adds **24 Prisma models**: 10 below, 5 lifecycle models in [10](10-pet-lifecycle.md#8-data-model-additions), and 9 in the extra features ([11](11-lost-stolen-and-qr-tag.md) `PetTag`, `PetTagScan`; [12](12-breeder-onboarding-and-score.md) `LitterDraft`, `Referral`; [13](13-health-guarantee-at-sale.md) `HealthCheck`; [14](14-city-licence-helper.md) `CityLicenceRule`, `PetLicence`; [15](15-adoption-and-shelters.md) `Shelter`, `AdoptionApplication`), **3 API modules**, and about **17 screens** across web and
admin. Each schema change ships as its own migration; each repo gets its own commits.

---

## 1. Database (`petzonic-api/prisma/schema.prisma`)

Conventions (existing): PascalCase models, snake_case tables via `@@map`, UUID PKs, money as
`Decimal(10,2)`, non-obvious decisions documented as prose comments in the schema.

### New models

| Model | Purpose | Key fields |
|---|---|---|
| `Pet` | Registry record; `UserPetProfile` migrates into it | `petCode` (unique Pet ID), `ringCode?` (unique), `speciesId`, `breedId`, `name`, `sex`, `dateOfBirth`, `colour`, `images[]`, `ownerId`, `breederId?`, `sireId?`, `damId?`, `litterId?`, `district`, `state`, `firstRegisteredState`, `kciNumber?`, `microchipNumber?` (unique), `ringNumber?` (unique), `nosePrintUrl?`, `idTrustLevel`, `visibility`, `status`, `diedAt?` |
| `Litter` | A birth event | `breederId`, `sireId?`, `damId`, `birthDate`, `count`, `speciesId`, `breedId` |
| `PetOwnershipTransfer` | Ownership history | `petId`, `fromUserId?`, `toUserId`, `reason`, `orderId?`, `transferredAt`, `acceptedAt?` |
| `PetMarkEvent` | Evidence of ring/chip link or scan | `petId`, `markType`, `markValue`, `evidenceUrl`, `recordedById`, `createdAt` |
| `RingBatch` | Leg-ring packs issued to verified breeders | `breederId`, `species`, `ringSizeMm`, `year`, `colour`, `firstPetCode`, `lastPetCode`, `issuedAt` |
| `PetShop` | Retail pet shop | `ownerId`, `name`, `address`, `district`, `state`, `latitude`, `longitude`, `speciesServed[]`, `timing`, `phone`, `isVerified`, `verifiedAt?`, `verifiedById?` |
| `VerificationApplication` | One workflow for all business types | `subjectType` (`SHOP`/`BREEDER`/`PHARMACY`/`PROVIDER`), `subjectId`, `status`, `documents[]`, `premisesCheckType`, `premisesCheckAt?`, `premisesNotes?`, `rejectionReason?`, `reviewedById?`, `verifiedAt?`, `expiresAt?`, `tier` (`BASIC`/`PLUS`) |
| `DataConsent` | Purpose-specific consent | `userId`, `purpose` (`ANALYTICS`/`BRAND_OFFERS`), `granted`, `grantedAt`, `withdrawnAt?`, `policyVersion` |
| `Campaign` | Brand campaign with embedded segment | `brandId`, `type`, `segment` (JSON: species, breeds, ageBands, districts, categories), `budget`, `startDate`, `endDate`, `impressions`, `clicks`, `sent`, `opened`, `status` |
| `PetInsightSnapshot` | Weekly aggregates | `period`, `speciesId`, `breedId?`, `district`, `state`, `ageBand`, `petCount`, `litterCount` — rows with `petCount < 50` are never written |

### New enums

`PetIdTrustLevel` (`SELF_DECLARED`, `MARKED`, `VERIFIED`) · `PetVisibility` (`PUBLIC_LIMITED`,
`PRIVATE`) · `PetStatus` (`ALIVE`, `DECEASED`, `RING_REMOVED`) · `TransferReason` (`SALE`, `GIFT`,
`REHOMING`, `ADMIN`) · `PetMarkType` (`RING`, `MICROCHIP`, `NOSE_PRINT`) · `VerificationSubject`
· `VerificationStatus` · `VerificationTier` · `ConsentPurpose` · `CampaignType` · `CampaignStatus`.

### Changes to existing models

| Model | Change |
|---|---|
| `PetListing` | Add `petId` FK (required for new listings; nullable for legacy rows) |
| `BreederProfile` | Add `latitude?`, `longitude?`; `isVerified` set only via `VerificationApplication` |
| `PharmacySellerProfile` | Verification moves onto the shared workflow (fields kept) |
| `ServiceProvider` | `isVerified` set only via the shared workflow |
| `Prescription` | Re-point from `UserPetProfile` to `Pet` after data migration |
| `VetConsultation`, `Booking`, `InsurancePolicy`, `LostFoundPost` | Add optional `petId` so each lands on the pet's timeline (see [10](10-pet-lifecycle.md#5-linking-existing-modules-to-the-pet-id)) |

### Constraints and indexes

- `@unique` on `Pet.petCode`, `Pet.ringCode`, `Pet.ringNumber`, `Pet.microchipNumber`.
- `@@index([district, speciesId, breedId])` on `Pet` for snapshots.
- `@@index([status, subjectType])` on `VerificationApplication` for the admin queue.
- `@@unique([userId, purpose])` on `DataConsent`.
- Ownership transfer on sale uses the existing guarded `updateMany` + `count === 0` idiom inside
  the escrow-release `$transaction`.

### Migration plan

1. `pet_registry_models` — create `Pet`, `Litter`, `PetOwnershipTransfer`, `PetMarkEvent`, enums.
2. `migrate_user_pet_profiles` — copy `UserPetProfile` rows into `Pet` (generate Pet IDs from the
   owner's state), re-point `Prescription`. Keep `UserPetProfile` read-only for one release.
3. `verification_and_shops` — `PetShop`, `VerificationApplication`, `RingBatch`, breeder lat/lng.
4. `consent_and_insights` — `DataConsent`, `Campaign`, `PetInsightSnapshot`.
5. `pet_listing_pet_id` — add nullable `petId` to `pet_listings`; enforce in the service layer.

## 2. API (`petzonic-api/src/modules/`)

All endpoints under `/api/v1`, standard `{ success, data, error, meta }` envelope, Zod schemas.

### `registry` (new module, mounted at `/api/v1/registry`)

`/api/v1/pets` is already taken by pet **listings** (`POST /pets` creates a listing,
`GET /pets/:id` reads one), so the registry gets its own prefix. Listing routes stay unchanged.

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | `/registry/pets` | authenticate | Register a pet; returns Pet ID |
| GET | `/registry/pets/mine` | authenticate | Owner's pets |
| GET | `/registry/:petCode` | optionalAuthenticate | Public pet page (limited fields, respects visibility) |
| GET | `/registry/:petCode/pedigree` | optionalAuthenticate | 3-generation tree |
| PATCH | `/registry/pets/:id` | authenticate (owner) | Update pet, visibility |
| POST | `/registry/pets/:id/mark` | authenticate | Link ring / chip / nose print; verified breeder or vet → `VERIFIED` |
| POST | `/registry/pets/:id/transfer` | authenticate (owner) | Start off-platform transfer |
| POST | `/registry/transfers/:id/accept` | authenticate | Receiver accepts |
| POST | `/registry/pets/:id/deceased` | authenticate (owner) | Retire Pet ID |
| POST | `/registry/litters` | authorize(BREEDER) | Register litter; creates child Pet IDs |
| POST | `/orders/:id/verify-pet` | authenticate (buyer) | Handover check (lives in `orders` module) |

Declare `/registry/pets/*` and `/registry/litters` before `/registry/:petCode` so the literal
paths win.

### `verification` (new module)

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | `/verification/apply` | authenticate | Create/submit application |
| GET | `/verification/me` | authenticate | Applicant status |
| GET | `/near-me` | public | Verified businesses near a point |
| POST | `/pet-shops` / PUT `/pet-shops/me` | authenticate | Shop profile |
| GET | `/admin/verification` | authorize(ADMIN) | Queue with filters |
| POST | `/admin/verification/:id/approve` \| `/reject` \| `/suspend` \| `/schedule-premises` | authorize(ADMIN) | Decisions (via `admin-verification.router.ts`) |
| POST | `/admin/rings/batches` | authorize(ADMIN) | Issue a ring batch |

### `insights` (new module)

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | `/admin/insights/population` etc. | authorize(ADMIN) | Snapshot dashboards |
| CRUD | `/admin/campaigns` | authorize(ADMIN) | Brand campaigns (`admin-campaigns.router.ts`) |
| GET | `/insights/pet-index` | public | Public quarterly index |

### `users` (existing)

`GET /users/me/consents`, `PUT /users/me/consents`.

### Scheduled jobs (new)

A scheduler already exists since 2026-09-22: `petzonic-maintenance-queue` in `src/lib/queue.ts`
uses BullMQ `upsertJobScheduler` and runs `auto-release-escrows` (hourly) and
`expire-abandoned-orders` (every 15 minutes). Add these jobs as new `task` values on that same
queue, following its pattern (stable scheduler ids, concurrency 1):

| Job | Schedule | Work |
|---|---|---|
| `insights.snapshot` | Weekly | Rebuild `PetInsightSnapshot` |
| `verification.expiry` | Daily | Expire, remind at 30/7 days |
| `breeder.score` | Weekly | Recalculate breeder scores ([12](12-breeder-onboarding-and-score.md#4-breeder-score)) |
| `licence.renewals` | Daily | City licence renewal reminders ([14](14-city-licence-helper.md)) |
| `adoption.followups` | Daily | 30-day and 6-month adopter check-ins ([15](15-adoption-and-shelters.md)) |
| `lifecycle.reminders` | Daily | Vaccine / deworming reminders, mark overdue doses |

Follow the house rule: if Redis is absent, log that jobs are disabled — never pretend they ran.

## 3. Web (`petzonic-web`)

| Route | Purpose |
|---|---|
| `/account/pets`, `/account/pets/new`, `/account/pets/[id]` | Owner registry (AuthGuard via `account/layout.tsx`) |
| `/registry/[petCode]` | Public pet page with family tree |
| `/breeders/[id]` | Breeder detail page (API already supports it) + litters |
| `/near-me` | Map + list |
| `/verification/apply` | Business verification stepper |
| `/account/settings` | Consent toggles |
| Order detail | Handover check block |

New client modules: `src/lib/registryApi.ts`, `verificationApi.ts`, `insightsApi.ts`. Shared
components: `components/trust/VerifiedBadge.tsx`, `PetIdPill.tsx`, `TrustLevelBadge.tsx`.

## 4. Admin (`petzonic-admin`)

| Route | Purpose |
|---|---|
| `/verification`, `/verification/[id]` | Queue and decision screen |
| `/registry` | Search by Pet ID, ring, chip, breeder; over-breeding flags |
| `/rings` | Issue and track ring batches |
| `/insights` | Snapshot dashboards |
| `/campaigns` | Brand campaigns |

Add a sidebar badge for pending verifications in `AdminSidebar.tsx`.

## 5. Tests

- API: Vitest + Supertest per new module against `petzonic_test`; security tests for
  visibility (no owner PII on public page), consent enforcement, cell suppression, and
  admin-only verification.
- Web: colocated component tests for badges, pet page, near-me; Playwright spec for the
  handover-check flow.
- Admin: first test setup for the verification queue (admin currently has zero tests).
