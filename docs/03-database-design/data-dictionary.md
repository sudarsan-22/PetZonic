# PetZonic — Data Dictionary

> **Version**: 1.0.0  
> **Date**: May 28, 2026

---

## Conventions

- **PK**: Primary Key (UUID v4, auto-generated)
- **FK**: Foreign Key (references another table's PK)
- **UK**: Unique constraint
- **NN**: Not Null
- All timestamps in UTC
- Monetary values stored as DECIMAL(10,2)
- All text fields trimmed on input
- Soft deletes via `status` field (not physical deletion)

---

## 1. Users Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Unique user identifier |
| phone | VARCHAR(15) | UK, Nullable | Phone number with country code (+91XXXXXXXXXX) |
| email | VARCHAR(255) | UK, Nullable | Email address (lowercase) |
| password_hash | VARCHAR(255) | Nullable | bcrypt hash (12 rounds), null for OTP-only users |
| status | ENUM | NN, Default: ACTIVE | ACTIVE, SUSPENDED, BANNED, DEACTIVATED |
| email_verified | BOOLEAN | NN, Default: false | Whether email is verified |
| phone_verified | BOOLEAN | NN, Default: false | Whether phone OTP was completed |
| last_login_at | TIMESTAMP | Nullable | Last successful login time |
| created_at | TIMESTAMP | NN, Auto | Account creation time |
| updated_at | TIMESTAMP | NN, Auto | Last profile update |

**Business Rules:**
- At least one of phone or email must be provided
- Phone format: E.164 format (+91XXXXXXXXXX for India)
- Email stored as lowercase
- Deactivated accounts: 30-day grace period before data deletion
- Banned users cannot re-register with same phone/email

---

## 2. Pet Listings Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Unique listing identifier |
| seller_id | UUID | FK→users, NN | User who created the listing |
| species_id | UUID | FK→pet_species, NN | Species category |
| breed_id | UUID | FK→pet_breeds, NN | Specific breed |
| title | VARCHAR(200) | NN | Listing title (e.g., "Golden Retriever Puppy - Vaccinated") |
| description | TEXT | NN, Min 20 chars | Detailed pet description |
| gender | ENUM | NN | MALE or FEMALE |
| age_months | INT | NN, Min 0, Max 360 | Pet age in months (max 30 years) |
| weight_kg | FLOAT | Nullable, Min 0.1 | Weight in kilograms |
| color | VARCHAR(50) | Nullable | Color description |
| price | DECIMAL(10,2) | NN, Min 100 | Price in INR |
| price_type | ENUM | NN, Default: FIXED | FIXED or NEGOTIABLE |
| status | ENUM | NN, Default: DRAFT | DRAFT, PENDING_REVIEW, ACTIVE, PAUSED, SOLD, EXPIRED, REJECTED |
| city | VARCHAR(100) | NN | Seller's city |
| state | VARCHAR(100) | NN | Seller's state |
| latitude | FLOAT | Nullable | Geo coordinate for proximity search |
| longitude | FLOAT | Nullable | Geo coordinate for proximity search |
| is_vaccinated | BOOLEAN | NN, Default: false | Whether pet has vaccination records |
| is_neutered | BOOLEAN | NN, Default: false | Whether pet is spayed/neutered |
| health_info | JSONB | Nullable | Additional health details (allergies, conditions) |
| view_count | INT | NN, Default: 0 | Number of detail page views |
| is_boosted | BOOLEAN | NN, Default: false | Whether listing is boosted/promoted |
| boost_expires_at | TIMESTAMP | Nullable | When boost period ends |
| expires_at | TIMESTAMP | NN | Auto-expire date (created_at + 60 days) |
| created_at | TIMESTAMP | NN, Auto | Listing creation time |
| updated_at | TIMESTAMP | NN, Auto | Last edit time |

**Business Rules:**
- Minimum 3 photos required before status can be ACTIVE
- Unverified sellers: listings go to PENDING_REVIEW
- Verified breeders: auto-approved (ACTIVE immediately)
- Expired listings hidden from search, seller notified 7 days before
- Sold listings remain in DB for history (not searchable)
- Price minimum ₹100 (prevents test/spam listings)
- Title cannot contain phone numbers or external links

---

## 3. Orders Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Unique order identifier |
| order_number | VARCHAR(20) | UK, NN | Human-readable order number (PZ-20260528-XXXX) |
| buyer_id | UUID | FK→users, NN | User who placed the order |
| address_id | UUID | FK→addresses, Nullable | Delivery address (null for pet meetups) |
| order_type | ENUM | NN | PRODUCT, PET, or SERVICE |
| status | ENUM | NN, Default: PENDING_PAYMENT | Current order status |
| subtotal | DECIMAL(10,2) | NN | Sum of item prices before fees/discounts |
| shipping_fee | DECIMAL(10,2) | NN, Default: 0 | Delivery charges |
| discount | DECIMAL(10,2) | NN, Default: 0 | Coupon/promo discount amount |
| tax | DECIMAL(10,2) | NN, Default: 0 | GST amount |
| total | DECIMAL(10,2) | NN | Final payable amount (subtotal + shipping - discount + tax) |
| coupon_code | VARCHAR(50) | Nullable | Applied coupon code |
| notes | VARCHAR(500) | Nullable | Special instructions from buyer |
| placed_at | TIMESTAMP | NN, Auto | Order placement time |
| delivered_at | TIMESTAMP | Nullable | Actual delivery timestamp |
| cancelled_at | TIMESTAMP | Nullable | Cancellation timestamp |

**Business Rules:**
- Order number format: PZ-{YYYYMMDD}-{4-digit random}
- Order cancellable only in CONFIRMED/PROCESSING status
- Pet orders: no shipping fee (meetup-based)
- Free shipping for product orders above ₹999
- GST calculated at 18% for products, 5% for pet food
- Total = subtotal + shipping_fee - discount + tax

---

## 4. Payments Table

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Internal payment identifier |
| order_id | UUID | FK→orders, NN | Associated order |
| user_id | UUID | FK→users, NN | User who made payment |
| razorpay_order_id | VARCHAR(100) | Nullable | Razorpay order ID (order_XXXXX) |
| razorpay_payment_id | VARCHAR(100) | UK, Nullable | Razorpay payment ID (pay_XXXXX) |
| amount | DECIMAL(10,2) | NN | Payment amount in INR |
| status | ENUM | NN, Default: CREATED | CREATED, AUTHORIZED, CAPTURED, FAILED, REFUNDED |
| method | ENUM | Nullable | UPI, CREDIT_CARD, DEBIT_CARD, NET_BANKING, WALLET, COD |
| metadata | JSONB | Nullable | Additional payment info (bank, card last4, UPI VPA) |
| created_at | TIMESTAMP | NN, Auto | Payment initiation time |
| updated_at | TIMESTAMP | NN, Auto | Last status change |

**Business Rules:**
- Never store full card numbers (only last 4 via Razorpay metadata)
- Failed payments: retry allowed within 30 minutes (same order)
- Multiple payment attempts per order allowed (only last successful counts)
- Refunds processed to original payment method
- COD available only for products < ₹5000

---

## 5. Reviews Table (Polymorphic)

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PK | Unique review identifier |
| reviewer_id | UUID | FK→users, NN | User who wrote the review |
| target_id | UUID | NN | ID of reviewed entity |
| target_type | ENUM | NN | SELLER, BREEDER, PRODUCT, SERVICE_PROVIDER, PET_LISTING |
| rating | INT | NN, 1-5 | Star rating |
| content | TEXT | Nullable, Max 2000 chars | Review text |
| photos | JSONB | Nullable | Array of photo URLs (max 5) |
| seller_response | TEXT | Nullable, Max 1000 chars | Seller's reply |
| is_verified_purchase | BOOLEAN | NN, Default: false | True if reviewer completed a transaction |
| helpful_count | INT | NN, Default: 0 | Number of "helpful" votes |
| created_at | TIMESTAMP | NN, Auto | Review posted time |
| updated_at | TIMESTAMP | NN, Auto | Last edit time |

**Business Rules:**
- One review per reviewer per target (unique: reviewer_id + target_id + target_type)
- Rating 1-5, required; content optional
- Editable within 7 days of creation
- Seller response: only one per review
- Verified purchase: system checks completed order before allowing review
- Photos: max 5, each < 5MB
- Auto-flag: reviews with profanity or suspicious patterns

---

## 6. Enum Value Definitions

### UserStatus
| Value | Description |
|-------|-------------|
| ACTIVE | Normal active account |
| SUSPENDED | Temporarily suspended (can be reactivated by admin) |
| BANNED | Permanently banned (cannot re-register) |
| DEACTIVATED | Self-deactivated by user (30-day deletion pending) |

### PetListingStatus
| Value | Description | Searchable |
|-------|-------------|:----------:|
| DRAFT | Created but not submitted | ❌ |
| PENDING_REVIEW | Submitted, awaiting moderation | ❌ |
| ACTIVE | Approved and visible | ✅ |
| PAUSED | Temporarily hidden by seller | ❌ |
| SOLD | Transaction completed | ❌ |
| EXPIRED | Passed 60-day expiry | ❌ |
| REJECTED | Rejected by moderation | ❌ |

### OrderStatus
| Value | Description | Next States |
|-------|-------------|-------------|
| PENDING_PAYMENT | Awaiting payment | CONFIRMED, CANCELLED |
| CONFIRMED | Payment received | PROCESSING, CANCELLED |
| PROCESSING | Being prepared | PACKED |
| PACKED | Ready for pickup | SHIPPED |
| SHIPPED | In transit | OUT_FOR_DELIVERY |
| OUT_FOR_DELIVERY | Last mile | DELIVERED |
| DELIVERED | Received by buyer | RETURNED |
| CANCELLED | Cancelled (any stage) | REFUNDED |
| RETURNED | Return initiated | REFUNDED |
| REFUNDED | Money returned | (terminal) |

---

## 7. JSON Field Schemas

### user_profiles.preferences
```json
{
  "interestedPetTypes": ["DOG", "CAT"],
  "interestedBreeds": ["uuid-1", "uuid-2"],
  "notificationPreferences": {
    "orders": true,
    "chat": true,
    "promotions": false,
    "petAlerts": true
  },
  "language": "en"
}
```

### pet_listings.health_info
```json
{
  "allergies": ["grain"],
  "conditions": [],
  "lastVetVisit": "2026-05-01",
  "dietType": "premium_dry_food",
  "specialNeeds": null
}
```

### service_providers.services_offered
```json
[
  { "name": "General Consultation", "price": 500, "duration": 30 },
  { "name": "Vaccination", "price": 800, "duration": 15 },
  { "name": "Home Visit", "price": 1200, "duration": 45 }
]
```

### service_providers.availability
```json
{
  "monday": { "start": "09:00", "end": "18:00", "slotDuration": 30 },
  "tuesday": { "start": "09:00", "end": "18:00", "slotDuration": 30 },
  "wednesday": null,
  "thursday": { "start": "09:00", "end": "18:00", "slotDuration": 30 },
  "friday": { "start": "09:00", "end": "18:00", "slotDuration": 30 },
  "saturday": { "start": "10:00", "end": "14:00", "slotDuration": 30 },
  "sunday": null
}
```

---

## 8. Seed Data Requirements

| Table | Seed Data |
|-------|-----------|
| pet_species | Dog, Cat, Bird, Fish, Rabbit, Hamster, Turtle, Snake, Horse |
| pet_breeds | ~200 popular breeds across species |
| product_categories | Food, Accessories, Toys, Grooming, Health, Housing (+ subcategories) |
| Admin user | Super admin account for initial access |
