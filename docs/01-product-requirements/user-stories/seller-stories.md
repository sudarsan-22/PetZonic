# User Stories — Seller & Broker

> **Role**: Individual Seller / Broker  
> **Description**: Individual selling own pet(s) or broker facilitating sales for multiple owners

---

## Onboarding & Verification

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-S01 | As a seller, I want to register as a seller so I can list pets for sale | Role selection during signup, seller dashboard appears after selection |
| US-S02 | As a seller, I want to complete KYC verification so buyers trust me | Upload Aadhaar/PAN, selfie verification, submission confirmation, 24-48hr review |
| US-S03 | As a seller, I want to know my verification status so I can track progress | Status shown: Pending → Under Review → Approved/Rejected (with reason) |
| US-S04 | As a seller, I want to set up my seller profile so buyers know about me | Display name, bio, location, contact preferences, profile photo |
| US-S05 | As a broker, I want to register as a broker so I can list multiple pets | Broker role selection, business details, volume listing capabilities enabled |
| US-S06 | As a broker, I want to add my business details so I appear professional | Business name, registration number (optional), address, operating since |

---

## Listing Management

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-S07 | As a seller, I want to create a pet listing so buyers can find my pet | Form: species, breed, age, gender, color, weight, price, description, health info |
| US-S08 | As a seller, I want to upload photos and video of my pet so buyers see it clearly | Upload: min 3, max 10 photos + 1 video (30s); auto-compress; preview before submit |
| US-S09 | As a seller, I want to attach vaccination records so buyers trust my pet's health | Upload PDF/images of vaccination certificates, checklist UI for common vaccines |
| US-S10 | As a seller, I want to set price as fixed or negotiable so buyers know my terms | Toggle: Fixed price / Negotiable; if negotiable, minimum acceptable price (hidden) |
| US-S11 | As a seller, I want to edit my listing after publishing so I can update details | Edit all fields, update photos, change price; changes reflected immediately |
| US-S12 | As a seller, I want to pause my listing temporarily so I stop receiving inquiries | Pause/Resume toggle; paused listings hidden from search, resume anytime |
| US-S13 | As a seller, I want to mark my pet as sold so buyers stop contacting me | "Mark as Sold" button; listing moves to "Sold" status, no longer searchable |
| US-S14 | As a seller, I want to delete my listing if I change my mind | Delete option with confirmation; listing permanently removed |
| US-S15 | As a seller, I want to get notified when my listing expires soon so I can renew | Push notification 7 days + 1 day before 60-day expiry; "Renew" button in notification |
| US-S16 | As a broker, I want to list multiple pets from one dashboard so I'm efficient | "My Listings" page: view all, filter by status, quick edit, bulk actions |
| US-S17 | As a broker, I want to duplicate a listing so I don't retype similar info | "Copy Listing" option: pre-fills form with existing data, change specifics |

---

## Buyer Interaction

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-S18 | As a seller, I want to receive chat messages from interested buyers | Push notification on new message, chat list shows unread count |
| US-S19 | As a seller, I want to respond to buyer questions quickly so I don't lose them | Chat opens from notification, quick reply templates available |
| US-S20 | As a seller, I want to share my location for meetup so the buyer can come | Share location button in chat, map view, address text |
| US-S21 | As a seller, I want to block abusive buyers so I'm not harassed | Block button in chat, blocked user can't message or view my listings |
| US-S22 | As a seller, I want to see how many views my listing gets so I know its performance | View count on listing card, views over time graph in analytics |

---

## Orders & Payments

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-S23 | As a seller, I want to get notified when a buyer initiates purchase | Push + SMS notification: "Buyer wants to purchase your [pet name]" |
| US-S24 | As a seller, I want to confirm or decline a pet order so I control the sale | Accept/Decline buttons on order notification; decline requires reason |
| US-S25 | As a seller, I want to coordinate delivery/meetup after order confirmation | Chat activated with buyer, meetup scheduling tools |
| US-S26 | As a seller, I want to receive payment after buyer confirms receipt | Auto-release from escrow after buyer confirms; settlement to bank account |
| US-S27 | As a seller, I want to see my earnings and payout history | Earnings page: total earned, pending, paid out; transaction history |
| US-S28 | As a seller, I want to add my bank account for payouts | Bank details form: account number, IFSC, account holder name; verification |
| US-S29 | As a seller, I want weekly payout to my bank account | Automated weekly settlement, payout summary notification |

---

## Promotion & Visibility

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-S30 | As a seller, I want to boost my listing so more buyers see it | "Boost" option: 7/14/30 day plans, payment flow, featured badge on listing |
| US-S31 | As a seller, I want to see which boost plan gives best results | Boost analytics: views before/after boost, inquiries received |

---

## Seller Analytics

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-S32 | As a seller, I want to see my overall performance metrics | Dashboard: total listings, active, sold, total views, total inquiries, rating |
| US-S33 | As a seller, I want to see my response rate so I can improve | Response rate %, average response time, compared to platform average |
| US-S34 | As a broker, I want to see sales analytics so I can optimize my business | Revenue report: total sales, by month, by breed, conversion rate |

---

## Account & Support

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-S35 | As a seller, I want to respond to buyer reviews so I can clarify issues | Reply button on received reviews; one reply per review; professional tone prompt |
| US-S36 | As a seller, I want to raise a dispute if buyer falsely claims issues | "Raise Dispute" option after buyer complaint; provide evidence; admin mediates |
| US-S37 | As a seller, I want to contact support if I face issues with the platform | In-app support ticket; category selection; response within 24hr SLA |
