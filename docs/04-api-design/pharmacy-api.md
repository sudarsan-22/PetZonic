# PetZonic — Pharmacy API

> **Base URL**: `/api/v1/pharmacy` (customer) · `/api/v1/admin/pharmacy` (admin)  
> **Source of truth**: `petzonic-api/src/modules/pharmacy/`  
> **Verified against source**: 2026-09-20

The pet pharmacy lets customers browse medicines, upload a veterinary prescription, and have
that prescription verified by an admin before prescription-only items can be fulfilled. It is
the step that follows a vet consultation in the customer journey.

---

## 1. Data Model

Six models back this module (all in `petzonic-api/prisma/schema.prisma`):

| Model | Table | Purpose |
|---|---|---|
| `PharmacyProduct` | `pharmacy_products` | Medicine-specific detail attached to a `Product` (1:1 via `Product.pharmacyDetails`) |
| `Prescription` | `prescriptions` | An uploaded or consultation-linked prescription awaiting/holding a verification decision |
| `PrescriptionItem` | `prescription_items` | Individual medicine lines on a prescription (`medicineName`, dosage, duration) |
| `OrderPrescription` | `order_prescriptions` | Links a prescription to an order at checkout |
| `PharmacySellerProfile` | `pharmacy_seller_profiles` | Licensed pharmacy seller, with license verification by an admin |
| `UserPetProfile` | `user_pet_profiles` | The customer's own pet records, used to scope a prescription to a specific animal |

### Enums

| Enum | Values |
|---|---|
| `PrescriptionStatus` | Verification lifecycle of a prescription |
| `PrescriptionSource` | How the prescription arrived (upload vs. linked consultation) |
| `PrescriptionRejectionReason` | Why an admin rejected a prescription |
| `OrderPrescriptionStatus` | State of the prescription attached to an order |
| `DosageForm` | Tablet, syrup, injection, etc. |
| `DrugSchedule` | Regulatory schedule classification of the medicine |

> Read the enum members directly from `schema.prisma` rather than copying them here — they
> change more often than this document does.

---

## 2. Customer Endpoints — `/api/v1/pharmacy`

### 2.1 Catalog (public)

| Method | Path | Description |
|---|---|---|
| `GET` | `/products` | List pharmacy products |
| `GET` | `/products/:slug` | Get one pharmacy product by slug |
| `GET` | `/search` | Search pharmacy products |
| `GET` | `/categories` | List pharmacy product categories |

### 2.2 Prescriptions (authenticated)

| Method | Path | Description |
|---|---|---|
| `GET` | `/prescriptions` | List the caller's own prescriptions |
| `GET` | `/prescriptions/:id` | Get one of the caller's prescriptions |
| `POST` | `/prescriptions/upload` | Upload a prescription document for verification |
| `POST` | `/prescriptions/link-consultation` | Attach a prescription issued during a vet consultation |

### 2.3 Pet profiles (authenticated)

| Method | Path | Description |
|---|---|---|
| `GET` | `/pets` | List the caller's pet profiles |
| `POST` | `/pets` | Create a pet profile |
| `PATCH` | `/pets/:id` | Update a pet profile |
| `DELETE` | `/pets/:id` | Delete a pet profile |

---

## 3. Admin Endpoints — `/api/v1/admin/pharmacy`

All routes require `authenticate` **and** `authorize("ADMIN")` — applied at the router level
in `admin-pharmacy.router.ts`.

| Method | Path | Description |
|---|---|---|
| `GET` | `/prescriptions` | List prescriptions awaiting or past review |
| `GET` | `/prescriptions/:id` | Inspect one prescription in detail |
| `POST` | `/prescriptions/:id/verify` | Record a verification decision (approve / reject) |
| `GET` | `/stats` | Pharmacy operational statistics |

---

## 4. Frontend Surfaces

| App | Route | Purpose |
|---|---|---|
| `petzonic-web` | `/pharmacy` | Pharmacy storefront |
| `petzonic-web` | `/pharmacy/products/[slug]` | Medicine detail page |
| `petzonic-admin` | `/pharmacy` | Prescription review queue |

---

## 5. Safety Note — AI Assistant Boundary

The AI shopping concierge deliberately **refuses to give dosage, diagnosis, or prescription
advice**. `petzonic-api/src/modules/ai-discovery/ai-discovery.service.ts` holds a
`PRESCRIPTION_OR_DIAGNOSIS_REGEX` guard that intercepts these requests before they reach the
model. Do not weaken or bypass that guard — prescription decisions belong to a licensed vet
and the admin verification flow, not to the assistant.
