# 07 — Privacy, Consent & DPDP Act Compliance

> **Status**: Proposed — not implemented · **Last Updated**: 2026-09-26  
> Back to [feature overview](README.md)

Using owner data for brand promotion is lawful under India's **Digital Personal Data Protection
Act, 2023** only with specific, separate, withdrawable consent. Consent is a **launch blocker**
for Phase 3, not a later add-on.

> This document records product rules, not legal advice. Legal review is required before
> Phase 3 goes live.

---

## 1. Consent model

| Purpose | Covers | Default | Where collected |
|---|---|---|---|
| Service (account terms) | Running PetZonic: orders, registry, verification | Required | Signup |
| `ANALYTICS` | Inclusion in aggregate insights and the Pet Index | Off (unticked) | Signup step + `/account/settings` |
| `BRAND_OFFERS` | Sponsored messages and samples from brands | Off (unticked) | Signup step + `/account/settings` |

- Plain-language notices in English, plus at least Tamil and Hindi at launch.
- Each change stores `policyVersion` and timestamp in `DataConsent`.
- **Withdraw any time**; withdrawal takes effect within 24 hours and removes the user from the
  next snapshot and every future campaign.

## 2. Purpose limitation

- Medical data (allergies, medical notes, prescriptions, diagnoses) is **never** used for
  insights or targeting. The existing Pino redaction list for medical fields stays intact.
- Brand-facing endpoints read only aggregate tables and campaign counters.
- Public pet pages show no owner name, phone, email or address — only district.

## 3. Minimisation and aggregation

- Minimum cell size **50 pets** for any published or reported figure.
- Segment dimensions are limited to species, breed, age band, district, state and purchase
  category.
- No export of row-level data to anyone outside PetZonic.

## 4. Retention and deletion

| Data | Retention |
|---|---|
| Pet record (registry) | Life of the pet; retired Pet ID kept to prevent reuse |
| Ownership transfer history | Life of the pet |
| Deleted user account | Personal fields erased; pet record keeps only species, breed, district and dates |
| Consent records | Kept for audit for the life of the account + 3 years |
| Verification documents | Validity period + 1 year, then deleted from storage |

## 5. Accountability

- Name a grievance officer and contact on `/privacy`.
- Log every campaign send, report download, consent change and verification decision in
  `AuditLog`.
- Security tests (API) for: consent enforcement, cell suppression, public-page PII exposure.
- Legal review of the privacy policy and brand contracts before Phase 3.
