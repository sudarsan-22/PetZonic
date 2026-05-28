# PetZonic — Pets API

> **Base**: `/api/v1/pets`

---

## GET /pets

List and search pet listings with filters.

**Auth**: Optional (auth provides personalized results)

### Query Parameters
| Param | Type | Description | Example |
|-------|------|-------------|---------|
| page | int | Page number (default: 1) | 1 |
| limit | int | Items per page (default: 20, max: 50) | 20 |
| species | string | Filter by species slug | dog |
| breed | string | Filter by breed slug | golden-retriever |
| gender | string | MALE or FEMALE | MALE |
| minPrice | number | Minimum price (INR) | 5000 |
| maxPrice | number | Maximum price (INR) | 50000 |
| minAge | int | Minimum age in months | 2 |
| maxAge | int | Maximum age in months | 12 |
| city | string | City name | bangalore |
| radius | int | Radius in km from user location | 25 |
| lat | float | User latitude (for distance sort) | 12.97 |
| lng | float | User longitude | 77.59 |
| vaccinated | boolean | Only vaccinated pets | true |
| priceType | string | FIXED or NEGOTIABLE | NEGOTIABLE |
| sortBy | string | Sort field | price |
| sortOrder | string | asc or desc | asc |
| q | string | Text search query | labrador puppy |

### Response (200)
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "Golden Retriever Puppy - Fully Vaccinated",
      "species": { "id": "uuid", "name": "Dog", "slug": "dog" },
      "breed": { "id": "uuid", "name": "Golden Retriever", "slug": "golden-retriever" },
      "gender": "MALE",
      "ageMonths": 3,
      "price": 25000,
      "priceType": "NEGOTIABLE",
      "city": "Bangalore",
      "state": "Karnataka",
      "distance": 5.2,
      "isVaccinated": true,
      "primaryImage": "https://media.petzonic.com/pets/uuid/medium/1.jpg",
      "viewCount": 142,
      "isBoosted": false,
      "seller": {
        "id": "uuid",
        "name": "Rahul S.",
        "avatarUrl": "...",
        "isVerified": true,
        "rating": 4.5
      },
      "createdAt": "2026-05-20T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 85,
    "totalPages": 5,
    "hasNext": true,
    "hasPrev": false
  }
}
```

---

## GET /pets/:id

Get full pet listing details.

**Auth**: Optional

### Response (200)
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Golden Retriever Puppy - Fully Vaccinated",
    "description": "Beautiful golden retriever puppy, 3 months old. Father is KCI registered champion. Very friendly and playful. All vaccinations up to date.",
    "species": { "id": "uuid", "name": "Dog", "slug": "dog" },
    "breed": { "id": "uuid", "name": "Golden Retriever", "slug": "golden-retriever" },
    "gender": "MALE",
    "ageMonths": 3,
    "weightKg": 4.5,
    "color": "Golden",
    "price": 25000,
    "priceType": "NEGOTIABLE",
    "status": "ACTIVE",
    "city": "Bangalore",
    "state": "Karnataka",
    "latitude": 12.97,
    "longitude": 77.59,
    "isVaccinated": true,
    "isNeutered": false,
    "healthInfo": {
      "allergies": [],
      "conditions": [],
      "lastVetVisit": "2026-05-15",
      "dietType": "premium_dry_food"
    },
    "viewCount": 142,
    "media": [
      { "id": "uuid", "url": "https://media.petzonic.com/...", "type": "IMAGE", "isPrimary": true },
      { "id": "uuid", "url": "https://media.petzonic.com/...", "type": "IMAGE", "isPrimary": false },
      { "id": "uuid", "url": "https://media.petzonic.com/...", "type": "VIDEO", "isPrimary": false }
    ],
    "vaccinations": [
      { "id": "uuid", "name": "Distemper", "date": "2026-04-01", "certificateUrl": "..." },
      { "id": "uuid", "name": "Parvovirus", "date": "2026-04-15", "certificateUrl": "..." }
    ],
    "seller": {
      "id": "uuid",
      "name": "Rahul Sharma",
      "avatarUrl": "...",
      "isVerified": true,
      "isBreeder": true,
      "rating": 4.5,
      "reviewCount": 23,
      "memberSince": "2025-03-15",
      "responseRate": 95,
      "totalListings": 12
    },
    "isSaved": false,
    "createdAt": "2026-05-20T10:00:00Z",
    "expiresAt": "2026-07-19T10:00:00Z"
  }
}
```

---

## POST /pets

Create a new pet listing.

**Auth**: Required (SELLER or BREEDER role)

### Request
```json
{
  "speciesId": "uuid",
  "breedId": "uuid",
  "title": "Golden Retriever Puppy - Fully Vaccinated",
  "description": "Beautiful golden retriever puppy, 3 months old...",
  "gender": "MALE",
  "ageMonths": 3,
  "weightKg": 4.5,
  "color": "Golden",
  "price": 25000,
  "priceType": "NEGOTIABLE",
  "city": "Bangalore",
  "state": "Karnataka",
  "latitude": 12.97,
  "longitude": 77.59,
  "isVaccinated": true,
  "isNeutered": false,
  "healthInfo": {
    "allergies": [],
    "dietType": "premium_dry_food"
  },
  "mediaIds": ["uuid-1", "uuid-2", "uuid-3"],
  "vaccinations": [
    { "name": "Distemper", "date": "2026-04-01", "certificateMediaId": "uuid" }
  ],
  "publishImmediately": true
}
```

### Validation Rules
- title: 10-200 characters, no phone numbers/links
- description: 20-2000 characters
- ageMonths: 0-360
- price: 100-10,000,000
- mediaIds: min 3, max 10 (must be pre-uploaded via /media/upload)
- publishImmediately: if true → PENDING_REVIEW (unverified) or ACTIVE (verified breeder)

### Response (201)
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "PENDING_REVIEW",
    "message": "Listing submitted for review. You'll be notified once approved."
  }
}
```

---

## PATCH /pets/:id

Update an existing listing.

**Auth**: Required (owner only)

### Request (partial update)
```json
{
  "price": 22000,
  "description": "Updated description..."
}
```

### Response (200)
```json
{
  "success": true,
  "data": { "id": "uuid", "message": "Listing updated" }
}
```

---

## PATCH /pets/:id/status

Change listing status.

**Auth**: Required (owner only)

### Request
```json
{
  "status": "PAUSED"
}
```

Allowed transitions:
- Owner: ACTIVE → PAUSED, PAUSED → ACTIVE, ACTIVE → SOLD
- Admin: Any → REJECTED

---

## DELETE /pets/:id

Delete a listing permanently.

**Auth**: Required (owner or ADMIN)

### Response (200)
```json
{
  "success": true,
  "data": { "message": "Listing deleted" }
}
```

---

## POST /pets/:id/boost

Boost listing visibility (paid feature).

**Auth**: Required (owner only)

### Request
```json
{
  "plan": "7_DAYS",
  "paymentId": "pay_razorpay_id"
}
```

Plans: `7_DAYS` (₹99), `14_DAYS` (₹179), `30_DAYS` (₹299)

---

## GET /pets/:id/similar

Get similar pet listings.

**Auth**: Optional

### Response (200)
Returns array of 6-10 similar listings based on breed, price range, and location.

---

## GET /pets/my-listings

Get current seller's own listings.

**Auth**: Required (SELLER/BREEDER)

### Query Parameters
| Param | Type | Description |
|-------|------|-------------|
| status | string | Filter by status (ACTIVE, PAUSED, SOLD, EXPIRED) |
| page | int | Page number |
| limit | int | Items per page |

---

## POST /pets/:id/report

Report a suspicious listing.

**Auth**: Required

### Request
```json
{
  "reason": "SCAM",
  "description": "This listing appears to be fake. Same photos found on another website."
}
```

Reasons: `SCAM`, `WRONG_INFO`, `INAPPROPRIATE`, `ANIMAL_CRUELTY`, `DUPLICATE`, `OTHER`
