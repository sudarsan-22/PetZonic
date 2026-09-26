# 14 — City Pet Licence Helper

> **Status**: Proposed — not implemented · **Last Updated**: 2026-09-26  
> Back to [feature overview](README.md)

Cities are starting to require pet licences and microchips — the Greater Chennai Corporation
made dog microchipping mandatory from October 2025, and Bengaluru has pet licensing. PetZonic
can **fill the licence details automatically from the Pet ID** and remind owners to renew. It
helps owners and opens partnerships with city corporations.

---

## 1. What the helper does

| Step | PetZonic does | Owner does |
|---|---|---|
| 1. Check | Tells the owner if their city needs a licence (city rules table) | — |
| 2. Prepare | Builds a licence pack: owner details, pet details, microchip number, rabies vaccination certificate (vet recorded), photo | Confirms details |
| 3. Apply | Opens the city's official portal with a checklist and copy buttons; later, submits directly where a city partners with PetZonic | Submits on the city portal and pays the city fee |
| 4. Record | — | Uploads the licence number and document |
| 5. Renew | Reminder before expiry (scheduled job) | Renews |

PetZonic never claims to issue a licence; the city does. The helper is clearly labelled as a
helper, not a government service.

## 2. City rules table (`CityLicenceRule`)

Admin-maintained, one row per city: species covered, microchip required (yes/no), rabies
certificate required, fee (as published by the city), validity period, official portal URL,
last checked date. Every value must be taken from the city's official notice and re-checked
every 6 months; the page shows "Rules last checked on …".

Launch cities to research first: Chennai, Bengaluru, Mumbai, then the next largest by
registered pets from the Pet Index.

## 3. Partnerships with city corporations

- **Import**: take chip numbers already implanted under city programmes into Pet IDs (with
  owner consent), so city-chipped dogs become PetZonic *Verified ID* pets.
- **Offer**: free licence dashboard for the corporation — licences prepared via PetZonic per
  ward, vaccination coverage, lost/stolen reports — using aggregated data only.
- **Later**: direct submission API where a city allows it.

## 4. Data model

| Change | Fields |
|---|---|
| `CityLicenceRule` (new) | `city`, `state`, `speciesIds[]`, `microchipRequired`, `rabiesCertRequired`, `feeText`, `validityMonths`, `portalUrl`, `lastCheckedAt` |
| `PetLicence` (new) | `petId`, `city`, `licenceNumber`, `issuedAt`, `expiresAt`, `documentUrl`, `status` |

Licence events also appear on the pet timeline (`LICENCE_ISSUED`, `LICENCE_RENEWED`).

## 5. API and UI

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | `/registry/pets/:id/licence-check` | owner | Is a licence needed, what is missing |
| GET | `/registry/pets/:id/licence-pack.pdf` | owner | Pre-filled pack |
| POST | `/registry/pets/:id/licences` | owner | Record licence number and document |
| CRUD | `/admin/city-licence-rules` | authorize(ADMIN) | Maintain rules |

UI: a "City licence" card on the owner pet page: green tick when valid, amber "Needed in
Chennai — 2 things missing: microchip, rabies certificate" with buttons that lead to
"Find a vet" and "Download licence pack".

## 6. Acceptance criteria

- [ ] The helper shows "Not required" when no rule exists for the owner's city.
- [ ] The licence pack includes only vet-recorded rabies vaccinations.
- [ ] Renewal reminder fires 30 days before `expiresAt`.
- [ ] Every city rule shows its last checked date.
