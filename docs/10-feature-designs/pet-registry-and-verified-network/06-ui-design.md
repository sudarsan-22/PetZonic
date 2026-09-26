# 06 — UI Design

> **Status**: Proposed — not implemented · **Last Updated**: 2026-09-26  
> Back to [feature overview](README.md)

Trust gets **one colour (teal) and one quiet badge style** across every screen; orange stays for
prices and buttons only. The design is clean and minimal and reuses the existing
`petzonic-web` design tokens.

---

## 1. Existing design foundation (from `petzonic-web/src/app/globals.css`)

| Token | Value | Role in this feature |
|---|---|---|
| `--primary` | `#ff8c00` | Prices, main buttons only |
| `--secondary` | `#14b8a6` | **Verified / trusted** — all trust badges |
| `--foreground` / `--text` | `#1e293b` | Body text |
| `--text-light` | `#64748b` | Neutral info, *Self-declared* text, *Marked* outline |
| `--muted` | `#f8fafc` | Pet ID pill background |
| `--border` | `#e2e8f0` | Card borders |
| `--success` / `--error` / `--warning` | `#22c55e` / `#ef4444` / `#f59e0b` | Handover result, suspensions only |
| Fonts | Inter (body), Poppins (headings) | Unchanged |
| Cards | `bg-white rounded-xl border border-border` | Unchanged |
| Icons | `lucide-react` | `BadgeCheck`, `ScanLine`, `Tag`, `MapPin`, `Copy` |

## 2. Design rules

- **Colour roles.** Teal = verified and trusted. Orange = price and main action. Slate =
  neutral info. Red and amber only for warnings.
- **No emoji or gradients on trust marks.** `PetCard.tsx` today shows three different badge
  styles (amber gradient with a trophy emoji for "Breeder Direct", dark "Pet Store", teal
  "Verified"). Replace them with the single system below.
- **Pet ID looks like an ID.** Monospace text in a `bg-muted` pill with a copy button.
- **One idea per card.** Name, one badge, distance or price. Details live on the detail page.
- **Mobile first.** 16 px side margins, one column under 640 px, filters in a bottom sheet.
- **Accessibility.** Badges always carry text, never icon-only; teal on white passes contrast
  at the `text-xs font-semibold` size used today; tap targets ≥ 44 px.

## 3. Badge system

| Badge | Meaning | Style (Tailwind) | Icon |
|---|---|---|---|
| PetZonic Verified | Business passed documents + premises check | `bg-secondary text-white rounded-full px-2 py-0.5 text-[11px] font-semibold` | `BadgeCheck` |
| Verified Plus | Paid tier of a verified business | Same pill + "Plus" in `text-teal-100` | `BadgeCheck` |
| Verified ID | Ring or chip scanned by a verified breeder or vet | `border border-secondary text-secondary rounded-full` | `ScanLine` |
| Marked | Ring or chip number entered by owner | `border border-border text-text-light rounded-full` | `Tag` |
| Self-declared | Photos only | Plain `text-xs text-text-light`, no pill | — |

Components to create in `petzonic-web/src/components/trust/`: `VerifiedBadge.tsx`,
`TrustLevelBadge.tsx`, `PetIdPill.tsx`.

## 4. Screens

| # | Screen | Route | Key elements |
|---|---|---|---|
| 1 | Pet card | (component) | Badge bottom-left of photo; grey line under breed: `Verified ID · PZ-TN-2026-000123` |
| 2 | Public pet page | `/registry/[petCode]` | Photo, name, breed/sex/age, Pet ID pill, trust badge; tabs **Overview · Family tree · History**; owner shown as "Owner in <district>" |
| 3 | Near me | `/near-me` | Desktop: list left, map right. Mobile: list + floating "Map" toggle. Chips: All · Shops · Breeders · Vets · Pharmacy; radius 1–50 km; "Open now" |
| 4 | My pets | `/account/pets` | Grid of pets + "Add pet" card. Add flow 3 steps: Basics → Identity mark → Visibility |
| 5 | Handover check | Order detail | One input "Enter ring or chip number"; match → teal tick + "Release payment"; mismatch → red note "Payment on hold, dispute opened" |
| 6 | Get verified | `/verification/apply` | 4-step stepper: Business → Documents → Premises check → Review; status timeline after submit |
| 7 | Breeder page | `/breeders/[id]` | Cover, farm name, badge, district, litters, pets available |
| 8 | Admin queue | admin `/verification` | Table with type/status filters; detail = documents left, checklist + decision right |
| 9 | Admin registry | admin `/registry` | Search by Pet ID / ring / chip / breeder; over-breeding flags |
| 10 | Owner pet page | `/account/pets/[id]` | Tabs Overview · Health · Timeline · Documents; "Next due" card; status chips Done (teal), Due soon (amber), Overdue (red), Planned (grey) — see [10](10-pet-lifecycle.md#10-ui) |
| 11 | Vet clinic screen | `/provider/pets/lookup` | Scan or enter Pet ID / ring / chip → summary → "Add vaccination" |
| 12 | Lost alert & tag page | `/t/[tagCode]`, pet page banner | Red banner "Reported lost", big "Message owner"; owner "Report lost / stolen" button — see [11](11-lost-stolen-and-qr-tag.md#8-ui) |
| 13 | Litter registration | `/breeder/litters/new` | 3 screens, camera-first, Tamil/Hindi/English switch, progress dots — see [12](12-breeder-onboarding-and-score.md) |
| 14 | Breeder score | Breeder card and page | Number 0–100 + label (Excellent/Good/Fair/Needs improvement), "How is this calculated?" link |
| 15 | Guarantee card | Order page | Countdown, "Book free vet check", "Report a health problem", insurance offer — see [13](13-health-guarantee-at-sale.md#7-ui) |
| 16 | City licence card | Owner pet page | Valid (teal) / needed with missing items (amber), "Download licence pack" — see [14](14-city-licence-helper.md) |
| 17 | Adopt | `/adopt`, shelter pages | Adoption listings with "Apply to adopt"; shelter badge — see [15](15-adoption-and-shelters.md) |

## 5. Wireframes

### Public pet page (mobile)

```
+-----------------------------------+
|  [        pet photo          ]    |
|                                   |
|  Bruno                            |
|  Labrador Retriever · Male · 8 mo |
|  [PZ-TN-2026-000123  copy]        |
|  (ScanLine) Verified ID           |
|                                   |
|  Overview | Family tree | History |
|  ---------------------------------|
|  Breeder   Green Paws Kennels  (v)|
|  Born      12 Jan 2026, Salem     |
|  Vaccines  Up to date             |
|  Lives in  Coimbatore, TN         |
+-----------------------------------+
```

### Family tree tab

```
 Grandparents        Parents           Pet
 [Sire's sire ]──┐
                 ├─[Sire  PZ-..]──┐
 [Sire's dam  ]──┘                │
                                  ├──[Bruno PZ-TN-2026-000123]
 [Dam's sire  ]──┐                │
                 ├─[Dam   PZ-..]──┘
 [Dam's dam   ]──┘
```
Each box is a small card (name, breed, trust badge); tap opens that pet's page. Unknown parents
show a dashed empty card "Not recorded".

### Near me (mobile)

```
+-----------------------------------+
|  Near me              [Map]       |
|  (All)(Shops)(Breeders)(Vets)     |
|  Within 5 km  v      [Open now]   |
|-----------------------------------|
|  Happy Tails Pet Shop             |
|  (v) PetZonic Verified            |
|  1.2 km · 4.6 * · Open till 9 pm |
|-----------------------------------|
|  Green Paws Kennels               |
|  (v) PetZonic Verified Plus       |
|  3.8 km · 4.8 * · Farm visits    |
+-----------------------------------+
```

### Handover check (order page)

```
+-----------------------------------+
|  Confirm it's the right pet       |
|  Enter the ring or chip number    |
|  [ PZTN26000123              ]    |
|  [        Check number        ]   |
|                                   |
|  ✓ Matches Bruno (PZ-TN-...123)   |   <- teal
|  [      Release payment       ]   |   <- orange
+-----------------------------------+
```

`(v)` = teal `BadgeCheck`. Names in wireframes are placeholders.

## 6. Empty, loading and error states

- Near me with no results: "No verified businesses within 5 km" + "Increase distance" button.
- Location denied: fall back to saved address district, with a small note.
- Private pet page: "This pet's owner has made the page private" — Pet ID still resolvable so
  buyers can confirm it exists.
- Follow the existing pattern: fixtures in `src/data/*.ts` as offline fallback and test data.
