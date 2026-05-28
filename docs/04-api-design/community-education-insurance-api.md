# PetZonic — Community, Education & Insurance API

> **Version**: 1.0.0  
> **Base URLs**:  
> - `/api/v1/community` — Forums & Discussions  
> - `/api/v1/education` — Content & Training  
> - `/api/v1/insurance` — Pet Insurance

---

# Part 1: Community & Discussion Forums

## 1. Forum Posts

### 1.1 List Forum Posts

```
GET /api/v1/community/posts
```

**Query Parameters**:

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| category | string | — | `breed_discussion`, `health_nutrition`, `training_tips`, `lost_and_found`, `buy_sell_advice`, `general` |
| species | string | — | Filter by species |
| sort | string | newest | `newest`, `popular`, `most_voted`, `most_replied` |
| q | string | — | Search posts |
| page | integer | 1 | Page |
| limit | integer | 20 | Per page |

**Response** `200 OK`:

```json
{
  "data": [
    {
      "id": "post_abc123",
      "category": "health_nutrition",
      "title": "Best food for a 3-month-old Golden Retriever?",
      "body": "Just got a new puppy and confused between Royal Canin and Drools...",
      "images": [],
      "species": "dog",
      "breed": "Golden Retriever",
      "tags": ["puppy-food", "golden-retriever"],
      "author": {
        "id": "user_123",
        "name": "Amit K.",
        "avatar": "...",
        "badges": ["verified_buyer"]
      },
      "stats": {
        "views": 234,
        "replies": 12,
        "upvotes": 45,
        "downvotes": 2
      },
      "isPinned": false,
      "createdAt": "2026-05-27T10:30:00Z"
    }
  ],
  "pagination": { "page": 1, "limit": 20, "total": 156, "totalPages": 8 }
}
```

---

### 1.2 Create Forum Post

```
POST /api/v1/community/posts
```

**Headers**: `Authorization: Bearer <token>`

**Body**:

```json
{
  "category": "health_nutrition",
  "title": "Best food for a 3-month-old Golden Retriever?",
  "body": "Just got a new puppy and confused between Royal Canin and Drools. My vet suggested...",
  "images": ["https://cdn.petzonic.com/forum/img1.jpg"],
  "species": "dog",
  "breed": "Golden Retriever",
  "tags": ["puppy-food", "golden-retriever"]
}
```

**Response** `201 Created`:

```json
{
  "id": "post_new123",
  "message": "Post created successfully"
}
```

---

### 1.3 Get Post Detail

```
GET /api/v1/community/posts/:id
```

Returns full post with first page of replies.

---

### 1.4 Update Post

```
PATCH /api/v1/community/posts/:id
```

Owner can edit within 24 hours. Shows "(edited)" badge.

---

### 1.5 Delete Post

```
DELETE /api/v1/community/posts/:id
```

Owner or admin only.

---

### 1.6 Vote on Post

```
POST /api/v1/community/posts/:id/vote
```

**Body**:

```json
{
  "value": 1
}
```

`value`: `1` (upvote), `-1` (downvote), `0` (remove vote)

---

### 1.7 Follow Post

```
POST /api/v1/community/posts/:id/follow
```

Toggle follow. Receive notifications on new replies.

---

## 2. Forum Replies

### 2.1 List Replies

```
GET /api/v1/community/posts/:postId/replies
```

**Query Parameters**: `page`, `limit`, `sort` (`newest`, `oldest`, `most_voted`)

---

### 2.2 Create Reply

```
POST /api/v1/community/posts/:postId/replies
```

**Body**:

```json
{
  "body": "I'd recommend Royal Canin Maxi Starter for the first few months...",
  "images": [],
  "parentId": null
}
```

`parentId` — set for nested reply (max 2 levels).

---

### 2.3 Vote on Reply

```
POST /api/v1/community/replies/:id/vote
```

Same as post voting.

---

## 3. Lost & Found

### 3.1 Create Lost/Found Post

```
POST /api/v1/community/lost-found
```

**Body**:

```json
{
  "type": "LOST",
  "petName": "Bruno",
  "species": "dog",
  "breed": "Beagle",
  "color": "Brown and white",
  "description": "Lost near Koramangala park around 6 PM. Wearing red collar with name tag.",
  "images": ["https://cdn.petzonic.com/lost/bruno1.jpg"],
  "lastSeenAt": "Koramangala 4th Block Park, Bangalore",
  "latitude": 12.9352,
  "longitude": 77.6245,
  "city": "Bangalore",
  "contactPhone": "+91-9876543210"
}
```

**Response** `201 Created` — Also triggers notification to users within 5km radius.

---

### 3.2 List Lost/Found Posts

```
GET /api/v1/community/lost-found
```

**Query Parameters**: `type` (LOST/FOUND), `city`, `species`, `lat`, `lng`, `radius`, `resolved`

---

### 3.3 Mark as Resolved

```
POST /api/v1/community/lost-found/:id/resolve
```

---

## 4. User Badges (Community)

| Badge | Criteria |
|-------|----------|
| `verified_buyer` | Completed at least one purchase |
| `verified_breeder` | KYC-verified breeder |
| `vet_verified` | Registered vet on platform |
| `top_contributor` | 50+ posts with positive karma |
| `helpful_expert` | 100+ replies with 500+ upvotes |

---

# Part 2: Educational Content & Training

## 5. Content Endpoints

### 5.1 List Educational Content

```
GET /api/v1/education/content
```

**Query Parameters**:

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| type | string | — | `article`, `video`, `guide` |
| category | string | — | `grooming`, `training`, `nutrition`, `health`, `breed_specific` |
| species | string | — | Filter by species |
| difficulty | string | — | `beginner`, `intermediate`, `advanced` |
| sort | string | newest | `newest`, `popular`, `most_liked` |
| page | integer | 1 | Page |
| limit | integer | 20 | Per page |

**Response** `200 OK`:

```json
{
  "data": [
    {
      "id": "edu_abc123",
      "type": "video",
      "title": "How to Groom Your Golden Retriever at Home",
      "slug": "groom-golden-retriever-home",
      "description": "Complete grooming tutorial covering brushing, bathing, nail trimming...",
      "thumbnailUrl": "https://cdn.petzonic.com/education/thumb_groom_golden.jpg",
      "videoUrl": "https://cdn.petzonic.com/education/groom_golden.mp4",
      "duration": 720,
      "category": "grooming",
      "species": "dog",
      "breed": "Golden Retriever",
      "difficulty": "beginner",
      "author": {
        "id": "user_groomer_1",
        "name": "Priya's Pet Spa",
        "avatar": "...",
        "badges": ["verified_provider"]
      },
      "stats": {
        "views": 5600,
        "likes": 342
      },
      "isPremium": false,
      "createdAt": "2026-05-10T08:00:00Z"
    }
  ],
  "pagination": { ... }
}
```

---

### 5.2 Get Content Detail

```
GET /api/v1/education/content/:slug
```

Returns full content with body/video URL, related content, author info.

---

### 5.3 Like Content

```
POST /api/v1/education/content/:id/like
```

Toggle like.

---

### 5.4 Create Content (Provider/Admin)

```
POST /api/v1/education/content
```

**Roles**: Admin, verified service providers, vets.

**Body**:

```json
{
  "type": "article",
  "title": "5 Signs Your Cat Needs Immediate Vet Attention",
  "body": "## Introduction\n\nCats are masters at hiding illness...",
  "thumbnailUrl": "...",
  "category": "health",
  "species": "cat",
  "difficulty": "beginner",
  "tags": ["cat-health", "emergency", "vet-visit"]
}
```

---

## 6. Video Courses

### 6.1 List Courses

```
GET /api/v1/education/courses
```

**Query Parameters**: `category`, `species`, `difficulty`, `premium` (true/false)

**Response** `200 OK`:

```json
{
  "data": [
    {
      "id": "course_001",
      "title": "Complete Puppy Training - From Zero to Obedient",
      "slug": "complete-puppy-training",
      "description": "12-video course covering basic commands...",
      "thumbnail": "...",
      "category": "training",
      "species": "dog",
      "difficulty": "beginner",
      "totalVideos": 12,
      "totalDuration": 5400,
      "isPremium": true,
      "price": 499,
      "author": {
        "name": "Ravi's Dog Academy",
        "avatar": "..."
      },
      "enrollmentCount": 234
    }
  ]
}
```

---

### 6.2 Enroll in Course

```
POST /api/v1/education/courses/:id/enroll
```

Free courses: instant enrollment. Premium courses: requires payment.

---

### 6.3 Get My Enrolled Courses

```
GET /api/v1/education/courses/enrolled
```

Returns courses with progress percentage.

---

### 6.4 Update Progress

```
POST /api/v1/education/courses/:courseId/progress
```

**Body**:

```json
{
  "contentId": "edu_video_5",
  "completed": true
}
```

---

## 7. Vet Telemedicine

### 7.1 Book Online Consultation

```
POST /api/v1/education/vet-consultation
```

**Body**:

```json
{
  "vetId": "user_vet_001",
  "type": "video",
  "petName": "Bruno",
  "petSpecies": "dog",
  "petBreed": "Golden Retriever",
  "symptoms": "Not eating properly for 2 days, seems lethargic",
  "attachments": ["https://cdn.petzonic.com/uploads/pet_photo.jpg"],
  "preferredDate": "2026-05-30",
  "preferredSlot": "10:00-10:30"
}
```

**Response** `201 Created`:

```json
{
  "id": "consult_abc123",
  "status": "scheduled",
  "scheduledAt": "2026-05-30T10:00:00+05:30",
  "videoRoomUrl": "https://meet.petzonic.com/room/consult_abc123",
  "amount": 500,
  "payment": {
    "razorpayOrderId": "order_xyz"
  }
}
```

---

### 7.2 List My Consultations

```
GET /api/v1/education/vet-consultation
```

---

### 7.3 Get Consultation Detail (with prescription)

```
GET /api/v1/education/vet-consultation/:id
```

Returns: consultation info, prescription, diagnosis, follow-up recommendation.

---

### 7.4 Vet: Complete Consultation

```
POST /api/v1/education/vet-consultation/:id/complete
```

**Body**:

```json
{
  "diagnosis": "Mild gastritis likely due to dietary change",
  "prescription": [
    {
      "medicine": "Pantoprazole 20mg",
      "dosage": "1 tablet",
      "frequency": "Before breakfast",
      "duration": "5 days"
    }
  ],
  "advice": "Switch back to previous food gradually over 5 days",
  "followUpDays": 7
}
```

---

### 7.5 Vet Q&A (Public Forum-style)

```
POST /api/v1/education/vet-qa
```

**Body**:

```json
{
  "question": "Is it safe to give curd to my 4-month-old Lab puppy?",
  "species": "dog",
  "breed": "Labrador",
  "images": []
}
```

Vets can respond within 24 hours. Becomes public for community benefit.

---

## 8. Feeding Calculator

```
GET /api/v1/education/feeding-calculator
```

**Query Parameters**:

| Param | Type | Description |
|-------|------|-------------|
| species | string | dog, cat, bird, fish |
| breed | string | Specific breed |
| ageMonths | integer | Age in months |
| weightKg | number | Weight in kg |
| activityLevel | string | low, moderate, high |

**Response** `200 OK`:

```json
{
  "recommendation": {
    "dailyCalories": 1200,
    "feedingFrequency": "2 times per day",
    "portionSize": "300g per meal",
    "foodType": "Adult dry food (large breed formula)",
    "notes": "At 3 years and 30kg, your Golden Retriever needs high-quality protein-rich food...",
    "suggestedProducts": [
      {
        "id": "prod_rc_maxi",
        "name": "Royal Canin Maxi Adult",
        "price": 6200
      }
    ]
  }
}
```

---

# Part 3: Pet Insurance

## 9. Insurance Plans

### 9.1 List Insurance Plans

```
GET /api/v1/insurance/plans
```

**Query Parameters**:

| Param | Type | Description |
|-------|------|-------------|
| species | string | Filter by species |
| coverageType | string | `basic`, `comprehensive`, `accident_only` |
| maxPremium | number | Monthly budget filter |
| partnerId | string | Filter by insurance company |

**Response** `200 OK`:

```json
{
  "data": [
    {
      "id": "plan_001",
      "partner": {
        "id": "partner_digit",
        "name": "Digit Insurance",
        "logo": "https://cdn.petzonic.com/partners/digit.png"
      },
      "name": "Comprehensive Pet Cover",
      "coverageType": "comprehensive",
      "speciesCovered": ["dog", "cat"],
      "coverageAmount": 500000,
      "premiumMonthly": 599,
      "premiumYearly": 5999,
      "deductible": 1000,
      "waitingPeriod": 14,
      "maxAge": 8,
      "coverageDetails": {
        "accidents": true,
        "illnesses": true,
        "surgery": true,
        "hospitalization": true,
        "thirdPartyLiability": true,
        "lostPet": false
      },
      "exclusions": [
        "Pre-existing conditions",
        "Cosmetic procedures",
        "Breeding-related issues"
      ]
    }
  ]
}
```

---

### 9.2 Get Plan Detail

```
GET /api/v1/insurance/plans/:id
```

Full plan info with complete T&C, claim process, documents required.

---

### 9.3 Compare Plans

```
GET /api/v1/insurance/plans/compare?ids=plan_001,plan_002,plan_003
```

Returns side-by-side comparison.

---

### 9.4 Premium Calculator

```
POST /api/v1/insurance/calculate-premium
```

**Body**:

```json
{
  "species": "dog",
  "breed": "Golden Retriever",
  "ageMonths": 24,
  "gender": "male",
  "coverageType": "comprehensive"
}
```

**Response**:

```json
{
  "plans": [
    {
      "planId": "plan_001",
      "partnerName": "Digit Insurance",
      "premiumMonthly": 599,
      "premiumYearly": 5999,
      "coverageAmount": 500000
    },
    {
      "planId": "plan_003",
      "partnerName": "Bajaj Allianz",
      "premiumMonthly": 749,
      "premiumYearly": 7499,
      "coverageAmount": 750000
    }
  ]
}
```

---

## 10. Policy Management

### 10.1 Purchase Insurance

```
POST /api/v1/insurance/policies
```

**Body**:

```json
{
  "planId": "plan_001",
  "petName": "Bruno",
  "petSpecies": "dog",
  "petBreed": "Golden Retriever",
  "petAge": 24,
  "petGender": "male",
  "paymentFrequency": "yearly",
  "paymentMethod": "razorpay"
}
```

**Response** `201 Created`:

```json
{
  "id": "policy_abc123",
  "policyNumber": "PZ-INS-2026-00456",
  "status": "active",
  "startDate": "2026-05-28",
  "endDate": "2027-05-28",
  "premiumPaid": 5999,
  "payment": {
    "razorpayOrderId": "order_ins_xyz"
  },
  "documentUrl": "https://cdn.petzonic.com/policies/PZ-INS-2026-00456.pdf"
}
```

---

### 10.2 List My Policies

```
GET /api/v1/insurance/policies
```

Returns all active + expired policies.

---

### 10.3 Get Policy Detail

```
GET /api/v1/insurance/policies/:id
```

---

### 10.4 Cancel Policy

```
POST /api/v1/insurance/policies/:id/cancel
```

Refund as per partner's cancellation policy.

---

## 11. Claims

### 11.1 File a Claim

```
POST /api/v1/insurance/claims
```

**Body**:

```json
{
  "policyId": "policy_abc123",
  "incidentDate": "2026-05-25",
  "description": "Bruno fell from stairs and fractured his front left leg. Visited vet immediately.",
  "claimAmount": 25000,
  "documents": [
    "https://cdn.petzonic.com/claims/vet_bill.pdf",
    "https://cdn.petzonic.com/claims/xray_report.pdf",
    "https://cdn.petzonic.com/claims/pet_photo.jpg"
  ]
}
```

**Response** `201 Created`:

```json
{
  "id": "claim_001",
  "claimNumber": "PZ-CLM-2026-00789",
  "status": "submitted",
  "message": "Claim submitted. Expected processing time: 5-7 business days."
}
```

---

### 11.2 Track Claim Status

```
GET /api/v1/insurance/claims/:id
```

---

### 11.3 List My Claims

```
GET /api/v1/insurance/claims
```

---

## 12. Insurance Recommendation (Post-Purchase)

```
GET /api/v1/insurance/recommend
```

**Query Parameters**: `species`, `breed`, `ageMonths`

Called after pet purchase to suggest relevant insurance plans.

---

## 13. Error Responses

| Status | Code | Module | Description |
|--------|------|--------|-------------|
| 400 | `POST_TOO_SHORT` | Community | Post body under minimum length |
| 403 | `FORUM_BANNED` | Community | User banned from forums |
| 404 | `POST_NOT_FOUND` | Community | Forum post doesn't exist |
| 404 | `CONTENT_NOT_FOUND` | Education | Educational content not found |
| 400 | `ALREADY_ENROLLED` | Education | Already enrolled in course |
| 400 | `VET_UNAVAILABLE` | Education | Vet not available at requested time |
| 404 | `PLAN_NOT_FOUND` | Insurance | Insurance plan not found |
| 400 | `PET_AGE_EXCEEDS_MAX` | Insurance | Pet too old for this plan |
| 400 | `POLICY_NOT_ACTIVE` | Insurance | Cannot claim on inactive policy |
| 409 | `DUPLICATE_CLAIM` | Insurance | Claim already filed for this incident |
