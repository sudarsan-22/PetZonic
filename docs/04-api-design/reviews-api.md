# PetZonic — Reviews API

> **Version**: 1.0.0  
> **Base URL**: `/api/v1/reviews`

---

## 1. Overview

The Reviews API handles ratings and reviews for pets/sellers, products, and service providers. Supports text reviews with photos, seller responses, helpful votes, and aggregated ratings.

---

## 2. Review Types

| Type | Target | Who Reviews | When |
|------|--------|-------------|------|
| `seller` | Seller profile | Buyer | After pet receipt confirmed |
| `product` | Product | Buyer | After order delivered |
| `service` | Service provider | Customer | After appointment completed |

---

## 3. Endpoints

### 3.1 Create Review

```
POST /api/v1/reviews
```

**Headers**: `Authorization: Bearer <token>`

**Body**:

```json
{
  "type": "seller",
  "targetId": "seller_abc123",
  "orderId": "ord_xyz789",
  "rating": 5,
  "title": "Excellent breeder, healthy puppy!",
  "comment": "The Golden Retriever puppy was exactly as described. Very healthy, well-socialized, and the breeder was extremely helpful with tips for first-time owners.",
  "images": [
    "https://cdn.petzonic.com/reviews/img1.jpg",
    "https://cdn.petzonic.com/reviews/img2.jpg"
  ],
  "tags": ["healthy-pet", "good-communication", "as-described"]
}
```

**Validation Rules**:
- `rating`: 1-5, required
- `title`: 5-100 chars, required
- `comment`: 20-2000 chars, required
- `images`: max 5, optional
- Can only review after order/booking is completed
- One review per order/booking

**Response** `201 Created`:

```json
{
  "id": "rev_new123",
  "type": "seller",
  "targetId": "seller_abc123",
  "rating": 5,
  "title": "Excellent breeder, healthy puppy!",
  "comment": "The Golden Retriever puppy was exactly as described...",
  "images": ["..."],
  "tags": ["healthy-pet", "good-communication", "as-described"],
  "reviewer": {
    "id": "user_123",
    "name": "Amit K.",
    "avatar": "https://cdn.petzonic.com/avatars/amit.jpg",
    "isVerifiedBuyer": true
  },
  "createdAt": "2026-05-28T14:30:00Z",
  "helpful": 0,
  "sellerResponse": null
}
```

---

### 3.2 Get Reviews for Target

```
GET /api/v1/reviews
```

**Query Parameters**:

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| type | string | — | `seller`, `product`, `service` |
| targetId | string | — | ID of seller/product/provider |
| rating | integer | — | Filter by star (1-5) |
| withImages | boolean | false | Only reviews with photos |
| sort | string | newest | `newest`, `oldest`, `helpful`, `rating_high`, `rating_low` |
| page | integer | 1 | Page |
| limit | integer | 10 | Per page |

**Response** `200 OK`:

```json
{
  "data": [
    {
      "id": "rev_001",
      "rating": 5,
      "title": "Excellent breeder!",
      "comment": "The Golden Retriever puppy was exactly as described...",
      "images": ["https://cdn.petzonic.com/reviews/img1.jpg"],
      "tags": ["healthy-pet", "as-described"],
      "reviewer": {
        "id": "user_123",
        "name": "Amit K.",
        "avatar": "...",
        "isVerifiedBuyer": true
      },
      "helpful": 12,
      "hasVotedHelpful": false,
      "sellerResponse": {
        "comment": "Thank you Amit! Glad to hear the puppy is doing well.",
        "respondedAt": "2026-05-29T09:00:00Z"
      },
      "createdAt": "2026-05-28T14:30:00Z"
    }
  ],
  "summary": {
    "average": 4.5,
    "total": 234,
    "distribution": {
      "5": 120,
      "4": 70,
      "3": 25,
      "2": 12,
      "1": 7
    },
    "topTags": [
      { "tag": "healthy-pet", "count": 89 },
      { "tag": "good-communication", "count": 67 },
      { "tag": "as-described", "count": 55 }
    ]
  },
  "pagination": { "page": 1, "limit": 10, "total": 234, "totalPages": 24 }
}
```

---

### 3.3 Edit Review

```
PATCH /api/v1/reviews/:id
```

**Rules**:
- Can only edit own review
- Editable within 30 days of creation
- Shows "edited" badge after modification

**Body** (partial):

```json
{
  "rating": 4,
  "comment": "Updated: puppy had minor health issue after a month but breeder helped resolve it."
}
```

---

### 3.4 Delete Review

```
DELETE /api/v1/reviews/:id
```

Only the reviewer or admin can delete. Soft-deletes the review.

---

### 3.5 Vote Helpful

```
POST /api/v1/reviews/:id/helpful
```

**Headers**: `Authorization: Bearer <token>`

Toggles helpful vote. Returns:

```json
{
  "helpful": 13,
  "hasVotedHelpful": true
}
```

---

### 3.6 Report Review

```
POST /api/v1/reviews/:id/report
```

**Body**:

```json
{
  "reason": "inappropriate",
  "details": "Contains abusive language"
}
```

**Reason Options**: `spam`, `inappropriate`, `fake`, `irrelevant`, `harassment`

---

## 4. Seller Response Endpoints

### 4.1 Respond to Review

```
POST /api/v1/reviews/:id/respond
```

**Headers**: `Authorization: Bearer <seller_token>`

**Rules**:
- Only the reviewed seller/provider can respond
- One response per review
- Can edit response once

**Body**:

```json
{
  "comment": "Thank you for your feedback! We're sorry about the issue and glad we could help resolve it."
}
```

---

### 4.2 Edit Response

```
PATCH /api/v1/reviews/:id/respond
```

**Body**:

```json
{
  "comment": "Updated response text..."
}
```

---

## 5. My Reviews (Authenticated User)

### 5.1 Reviews I've Written

```
GET /api/v1/reviews/mine
```

---

### 5.2 Pending Reviews (Orders/Bookings awaiting review)

```
GET /api/v1/reviews/pending
```

**Response** `200 OK`:

```json
{
  "data": [
    {
      "type": "seller",
      "orderId": "ord_123",
      "targetId": "seller_abc",
      "targetName": "Happy Paws Kennel",
      "orderDate": "2026-05-20T10:00:00Z",
      "daysRemaining": 25,
      "pet": "Golden Retriever Puppy"
    }
  ]
}
```

Reviews must be submitted within 30 days of order completion.

---

## 6. Rating Aggregation

Ratings are recalculated asynchronously via a background job whenever a new review is submitted:

```
Rating = (Σ all ratings) / count
```

Weighted display (for search ranking):
```
Weighted = (avg_rating × review_count + 3.5 × 10) / (review_count + 10)
```

This prevents sellers with 1 review of 5 stars from outranking those with 100 reviews averaging 4.5.

---

## 7. Review Tags (Predefined)

### Seller Reviews
| Tag | Display |
|-----|---------|
| `healthy-pet` | Healthy Pet |
| `as-described` | As Described |
| `good-communication` | Good Communication |
| `responsive` | Quick Response |
| `fair-price` | Fair Price |
| `well-socialized` | Well Socialized |
| `documentation-provided` | Docs Provided |

### Product Reviews
| Tag | Display |
|-----|---------|
| `good-quality` | Good Quality |
| `value-for-money` | Value for Money |
| `fast-delivery` | Fast Delivery |
| `well-packaged` | Well Packaged |
| `as-expected` | As Expected |

### Service Reviews
| Tag | Display |
|-----|---------|
| `professional` | Professional |
| `gentle-handling` | Gentle with Pet |
| `on-time` | On Time |
| `clean-facility` | Clean Facility |
| `knowledgeable` | Knowledgeable |

---

## 8. Error Responses

| Status | Code | Description |
|--------|------|-------------|
| 400 | `ALREADY_REVIEWED` | Order/booking already has a review |
| 400 | `ORDER_NOT_COMPLETED` | Cannot review before completion |
| 400 | `REVIEW_WINDOW_EXPIRED` | Past 30-day review window |
| 403 | `NOT_REVIEW_OWNER` | Cannot edit/delete others' reviews |
| 404 | `REVIEW_NOT_FOUND` | Review doesn't exist |
| 403 | `NOT_TARGET_OWNER` | Cannot respond to review of another seller |
