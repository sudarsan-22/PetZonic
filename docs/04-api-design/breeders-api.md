# PetZonic — Breeders & District Discovery API

> **Base URL**: `/api/v1/breeders`  
> **Source of truth**: `petzonic-api/src/modules/breeders/`  
> **Verified against source**: 2026-09-26

This module exists to close PetZonic's founding gap: a buyer who does not already know the
local pet trade has no way to find the breeder down the street, and defaults to a shop at a
higher price. Breeder profiles are district-scoped so that buyers can browse by locality the
way the existing district-level WhatsApp and Facebook breeder groups work offline.

> **Status (2026-09-20)**: implemented end to end — database, API, and web UI.
> The frontend lives at `petzonic-web/src/app/breeders/page.tsx` (~520 lines) with its API
> client at `petzonic-web/src/lib/breedersApi.ts`. It loads the district list, shows a breeder
> count per district, and supports district filtering via `?district=` query params.
>
> This feature was built rapidly on 2026-09-19/20; expect it to keep changing. Re-check the
> route tree before documenting its UI in detail.

---

## 1. Data Model

### `BreederProfile` → `breeder_profiles`

One profile per user (`userId` is `@unique`, cascade-deleted with the user).

| Field | Type | Notes |
|---|---|---|
| `farmName` | `VarChar(150)` | Required |
| `tagline` | `VarChar(255)?` | Optional short descriptor |
| `district` | `VarChar(100)` | Required — **indexed**, drives district discovery |
| `state` | `VarChar(100)` | Required — **indexed** |
| `experienceYears` | `Int` | Defaults to `1` |
| `specializationBreeds` | `String[]` | Defaults to empty array |
| `isFarmVisitAllowed` | `Boolean` | Defaults to `true` |
| `farmAddress` | `VarChar(300)?` | Optional |
| `aboutFarm` | `Text?` | Optional long-form description |
| `registrationNumber` | `VarChar(100)?` | Optional breeder registration id |
| `establishedYear` | `Int?` | Optional |
| `avatarUrl` / `coverImageUrl` | `VarChar(500)?` | Optional imagery |
| `isVerified` | `Boolean` | Defaults to **`false`** — admin-granted (since 2026-09-22) |

> **Verification (updated 2026-09-22, api `c3bfaeb`).** `isVerified` now defaults to `false`,
> and migration `20260922010000_breeder_verification_admin_granted` reset every existing
> profile to unverified, so the badge can no longer be self-assigned at signup.
> **Remaining gap:** there is still no admin endpoint or admin screen that grants it — it can
> only be set directly in the database today.

### `PetListing.district`

Migration `20260920011500_breeder_profile_and_district` also added a nullable
`district VARCHAR(100)` column to `pet_listings`, with a composite index
`pet_listings_district_status_idx` on `(district, status)` for district-filtered listing
queries. `district` is accepted as an optional filter by `pets.schema.ts`.

---

## 2. Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| `GET` | `/breeders` | Public | List breeder profiles |
| `GET` | `/breeders/districts` | Public | List districts that have breeders — powers district browse |
| `GET` | `/breeders/me` | Authenticated | Get the caller's own breeder profile |
| `PUT` | `/breeders/me` | Authenticated | Create or update the caller's breeder profile (upsert) |
| `GET` | `/breeders/:id` | Public | Get one breeder profile by id |

Mounted at `petzonic-api/src/app.ts` → `app.use("/api/v1/breeders", breedersRouter)`.

---

## 3. Frontend

| App | Route | Purpose |
|---|---|---|
| `petzonic-web` | `/breeders` | Breeder directory with district browse, counts per district, and `?district=` filtering (SEO metadata via `breeders/layout.tsx`) |
| `petzonic-web` | `/pets?sellerType=BREEDER` | Seller-type filter (`ALL`/`BREEDER`/`SHOP`/`INDIVIDUAL`), breeder badge on pet cards, breeder trust card |
| AI concierge | `targetTab: /breeders?state=…` | Breeder intent routes the chat to the breeder hub (since 2026-09-23) |

Client: `petzonic-web/src/lib/breedersApi.ts`.

---

## 4. Known Gaps

| Gap | Impact |
|---|---|
| No admin action to grant `isVerified` | Every breeder shows as unverified until an admin tool exists |
| No individual breeder detail page | `GET /breeders/:id` exists but no `/breeders/[id]` route consumes it |
| No breeder profile self-service page | `PUT /breeders/me` exists but no seller-facing form uses it |
| `PetListing.district` is nullable and not backfilled | Existing listings have no district, so district filtering silently excludes them |
| No link between `BreederProfile.district` and `PetListing.district` | The two are set independently and can disagree |
