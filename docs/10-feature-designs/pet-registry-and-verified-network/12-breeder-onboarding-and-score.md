# 12 — Easy Litter Registration, Breeder Referrals & Breeder Score

> **Status**: Proposed — not implemented · **Last Updated**: 2026-09-26  
> Back to [feature overview](README.md)

Home breeders are the group PetZonic most wants to capture, and many live on WhatsApp more than
on websites. Litter registration must take **3 steps on a phone**, in Tamil, Hindi or English,
with a WhatsApp link to finish. Free rings reward the first litter and every referred breeder. A
**breeder score**, calculated from registry records, then shows buyers who breeds well.

---

## 1. 3-step litter registration

| Step | What the breeder does | Notes |
|---|---|---|
| 1. Parents | Pick the mother (and father if known) from their pets, or add them in one line (breed + photo) | Parents get Pet IDs too |
| 2. Babies | Birth date, how many, one camera photo of the litter | Each baby gets a Pet ID; names optional |
| 3. Marks | Birds: link rings (type or scan the code). Dogs/cats: "chip later" allowed | 14-day reminder to link rings |

Design rules:
- Opens straight from a WhatsApp link, logs in by **OTP** (existing), no password.
- Camera-first photo capture; images compressed on the phone before upload.
- Big buttons, one question per screen, progress dots, works on slow mobile data.
- Language switch Tamil / Hindi / English at the top (web i18n does not exist yet — this flow is
  the first to need it).
- Saves a draft after every step; the breeder can finish later from a WhatsApp reminder.

## 2. WhatsApp as the front door

| Stage | How |
|---|---|
| Launch | **Click-to-chat links** (`wa.me`) and share buttons: "Register your litter" link in the PetZonic WhatsApp number's replies, in breeder groups, on ring packs (QR) |
| Later | WhatsApp Business Platform via an Indian WhatsApp solution provider: reminders ("link rings for your litter"), litter-registered confirmation with the Pet IDs, and a registration link that logs the breeder in with a one-time token |

Every WhatsApp template message needs opt-in and Meta template approval; until that exists,
SMS (MSG91) and push carry the same messages. The provider integration must follow the house
rule: unconfigured → report "WhatsApp not configured" and fall back, never pretend to send.

## 3. Free rings and referrals

| Trigger | Reward |
|---|---|
| First litter registered by a new breeder | Free ring pack for that litter (up to 10 rings) |
| Referred breeder registers their first litter | Referrer gets a free ring pack of 10 |
| Referred breeder becomes PetZonic Verified | Referrer gets 1 month of Verified Plus |

Rules: one referral code per breeder; rewards granted only after the referred breeder's litter is
registered with real photos and passes a quick admin check; maximum 10 rewarded referrals per
breeder per year; self-referrals (same phone, device or address) are blocked.

## 4. Breeder score

A score from **0 to 100**, shown on breeder pages and cards once a breeder has at least
**3 litters or 10 sold pets** (before that: "New breeder").

| Signal | Weight | Source |
|---|---|---|
| Pets alive and healthy at 1 year | 30% | Registry status + health passport |
| Buyer reviews | 25% | `Review` on the breeder's sales |
| Healthy breeding frequency (≤ 2 litters per dam per 12 months) | 15% | `Litter` |
| Vaccinations done before sale (vet recorded) | 15% | `PetVaccination` |
| Handover checks passed first time | 10% | `verify-pet` results |
| Upheld disputes and health-guarantee claims (penalty) | −5% each, capped | `Dispute` |

- Recalculated weekly by a scheduled job (`breeder.score`).
- Shown as a number with a plain label: 85–100 "Excellent", 70–84 "Good", 50–69 "Fair", below
  50 "Needs improvement" (below 50 also removes Verified Plus ranking).
- The breeder page explains how the score is made — no hidden formula, so breeders know how to
  improve it.
- Hard to fake: every signal comes from registry records, vet entries and paid orders, not from
  what the breeder types.

## 5. Data model

| Change | Fields |
|---|---|
| `LitterDraft` (new) | `breederId`, `step`, `data` (JSON), `updatedAt` |
| `Referral` (new) | `referrerId`, `referredUserId`, `code`, `status` (`PENDING`/`QUALIFIED`/`REWARDED`/`REJECTED`), `rewardType`, `createdAt` |
| `BreederProfile` | `referralCode` (unique), `score?`, `scoreLabel?`, `scoreUpdatedAt?`, `preferredLanguage` |
| `RingBatch` | `source` (`PURCHASE`/`FREE_FIRST_LITTER`/`REFERRAL`) |

## 6. API

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST/PUT | `/registry/litters/draft` | authorize(BREEDER) | Save draft per step |
| POST | `/registry/litters` | authorize(BREEDER) | Submit (already planned) |
| GET | `/breeders/me/referrals` | authorize(BREEDER) | Code, referred breeders, rewards |
| GET | `/breeders/:id` | public | Now includes `score`, `scoreLabel` |
| GET | `/admin/referrals` | authorize(ADMIN) | Review and approve rewards |

## 7. Acceptance criteria

- [ ] A breeder can register a litter in 3 screens on a 360 px wide phone.
- [ ] A draft survives closing the browser and reopens from the reminder link.
- [ ] First-litter free rings are issued once per breeder.
- [ ] A self-referral (same phone or device) is rejected.
- [ ] Breeder score is hidden until 3 litters or 10 sold pets, then updates weekly.
