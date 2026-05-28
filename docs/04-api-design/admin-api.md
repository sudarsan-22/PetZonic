# PetZonic — Admin API

> **Version**: 1.0.0  
> **Base URL**: `/api/v1/admin`

---

## 1. Overview

The Admin API provides endpoints for platform management including dashboard analytics, user management, content moderation, order management, and financial reporting. All endpoints require `admin` or `super_admin` role.

---

## 2. Authentication & Authorization

All admin endpoints require:
```
Authorization: Bearer <admin_jwt_token>
X-Admin-MFA: <totp_code>  (for sensitive operations)
```

**Roles**:
| Role | Access |
|------|--------|
| `super_admin` | Full access, can manage other admins |
| `admin` | Full moderation, user mgmt, orders |
| `moderator` | Listing moderation, basic user view |
| `support` | Order inquiries, disputes, read-only |
| `finance` | Revenue reports, payouts, refunds |

---

## 3. Dashboard

### 3.1 Get Dashboard Stats

```
GET /api/v1/admin/dashboard
```

**Query Parameters**:

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| period | string | today | `today`, `week`, `month`, `year`, `custom` |
| from | string | — | Start date (YYYY-MM-DD) for custom |
| to | string | — | End date for custom |

**Response** `200 OK`:

```json
{
  "period": "today",
  "metrics": {
    "revenue": {
      "total": 245600,
      "products": 180000,
      "pets": 55000,
      "services": 10600,
      "change": 12.5
    },
    "orders": {
      "total": 89,
      "products": 72,
      "pets": 12,
      "services": 5,
      "change": 8.3
    },
    "users": {
      "newRegistrations": 45,
      "activeToday": 1230,
      "totalUsers": 15670,
      "change": 3.2
    },
    "listings": {
      "newToday": 23,
      "totalActive": 890,
      "pendingModeration": 12,
      "reported": 3
    },
    "support": {
      "openDisputes": 5,
      "pendingKyc": 8,
      "pendingListings": 12
    }
  },
  "charts": {
    "revenueTimeline": [
      { "date": "2026-05-22", "amount": 198000 },
      { "date": "2026-05-23", "amount": 215000 }
    ],
    "ordersByCategory": [
      { "category": "Dog Food", "count": 34 },
      { "category": "Cat Accessories", "count": 18 }
    ],
    "topSellingProducts": [
      { "id": "prod_1", "name": "Royal Canin Maxi", "quantity": 45 }
    ]
  }
}
```

---

## 4. User Management

### 4.1 List Users

```
GET /api/v1/admin/users
```

**Query Parameters**:

| Param | Type | Description |
|-------|------|-------------|
| role | string | `buyer`, `seller`, `breeder`, `provider` |
| status | string | `active`, `suspended`, `banned` |
| verified | boolean | KYC verified status |
| q | string | Search name, email, phone |
| sort | string | `newest`, `oldest`, `name`, `orders` |
| page | integer | Page number |
| limit | integer | Per page |

---

### 4.2 Get User Detail

```
GET /api/v1/admin/users/:id
```

Returns full user profile, order history, listings, KYC status, activity log, reports against them.

---

### 4.3 Suspend User

```
POST /api/v1/admin/users/:id/suspend
```

**Body**:

```json
{
  "reason": "Multiple reports of selling sick pets",
  "duration": "30d",
  "notifyUser": true
}
```

---

### 4.4 Ban User

```
POST /api/v1/admin/users/:id/ban
```

**Body**:

```json
{
  "reason": "Fraudulent activity - selling stolen pets",
  "permanent": true,
  "notifyUser": true,
  "cancelActiveOrders": true,
  "deactivateListings": true
}
```

---

### 4.5 Reinstate User

```
POST /api/v1/admin/users/:id/reinstate
```

---

## 5. KYC Management

### 5.1 KYC Queue

```
GET /api/v1/admin/kyc
```

**Query Parameters**:

| Param | Type | Description |
|-------|------|-------------|
| status | string | `pending`, `approved`, `rejected` |
| type | string | `seller`, `breeder`, `provider` |

---

### 5.2 Review KYC

```
POST /api/v1/admin/kyc/:id/review
```

**Body**:

```json
{
  "decision": "approved",
  "notes": "All documents verified, FSSAI license valid"
}
```

Or rejection:

```json
{
  "decision": "rejected",
  "reason": "Document expired - please upload current Aadhaar",
  "resubmitAllowed": true
}
```

---

## 6. Listing Moderation

### 6.1 Moderation Queue

```
GET /api/v1/admin/moderation/listings
```

**Query Parameters**:

| Param | Type | Description |
|-------|------|-------------|
| status | string | `pending`, `reported`, `flagged` |
| species | string | Filter by species |
| sort | string | `oldest_first` (FIFO), `reports_desc` |

---

### 6.2 Approve Listing

```
POST /api/v1/admin/moderation/listings/:id/approve
```

**Body**:

```json
{
  "notes": "Verified - all photos genuine, health records match"
}
```

---

### 6.3 Reject Listing

```
POST /api/v1/admin/moderation/listings/:id/reject
```

**Body**:

```json
{
  "reason": "prohibited_species",
  "message": "We do not allow sale of wild/exotic species without proper CITES documentation.",
  "canResubmit": false
}
```

**Rejection Reasons**:
- `poor_photos` — Blurry/insufficient photos
- `incomplete_info` — Missing required details
- `prohibited_species` — Banned species
- `suspected_fraud` — Possible scam
- `health_concerns` — Animal appears unhealthy
- `price_unrealistic` — Price seems too low/high
- `duplicate` — Already listed

---

### 6.4 Flag Listing

```
POST /api/v1/admin/moderation/listings/:id/flag
```

Marks for re-review without removing.

---

## 7. Order Management

### 7.1 List All Orders

```
GET /api/v1/admin/orders
```

**Query Parameters**:

| Param | Type | Description |
|-------|------|-------------|
| status | string | Order status filter |
| type | string | `product`, `pet`, `service` |
| dateFrom | string | Start date |
| dateTo | string | End date |
| minAmount | number | Minimum order value |
| userId | string | Filter by user |
| sellerId | string | Filter by seller |

---

### 7.2 Force Status Change

```
POST /api/v1/admin/orders/:id/status
```

**Body**:

```json
{
  "status": "cancelled",
  "reason": "Seller unresponsive for 7 days",
  "refundBuyer": true,
  "penalizeSeller": true
}
```

---

### 7.3 Process Refund

```
POST /api/v1/admin/orders/:id/refund
```

**Body**:

```json
{
  "amount": 5000,
  "type": "full",
  "reason": "Pet health issue within guarantee period",
  "source": "escrow"
}
```

---

## 8. Dispute Management

### 8.1 List Disputes

```
GET /api/v1/admin/disputes
```

---

### 8.2 Get Dispute Detail

```
GET /api/v1/admin/disputes/:id
```

Returns: dispute info, order, both parties' evidence, chat history, timeline.

---

### 8.3 Resolve Dispute

```
POST /api/v1/admin/disputes/:id/resolve
```

**Body**:

```json
{
  "resolution": "favor_buyer",
  "action": "full_refund",
  "amount": 15000,
  "sellerPenalty": "warning",
  "adminNotes": "Buyer provided vet report showing pre-existing condition not disclosed",
  "notifyBothParties": true
}
```

**Resolution Options**: `favor_buyer`, `favor_seller`, `partial_refund`, `mutual_agreement`

---

## 9. Product & Category Management

### 9.1 Product CRUD

```
POST   /api/v1/admin/products
GET    /api/v1/admin/products
GET    /api/v1/admin/products/:id
PATCH  /api/v1/admin/products/:id
DELETE /api/v1/admin/products/:id
```

### 9.2 Category CRUD

```
POST   /api/v1/admin/categories
GET    /api/v1/admin/categories
PATCH  /api/v1/admin/categories/:id
DELETE /api/v1/admin/categories/:id
PUT    /api/v1/admin/categories/reorder
```

---

## 10. Financial Reports

### 10.1 Revenue Report

```
GET /api/v1/admin/reports/revenue
```

**Query Parameters**: `period`, `from`, `to`, `groupBy` (`day`, `week`, `month`)

**Response**:

```json
{
  "summary": {
    "grossRevenue": 2450000,
    "commissions": 245000,
    "productSales": 1800000,
    "serviceFees": 50000,
    "refunds": 35000,
    "netRevenue": 2060000
  },
  "timeline": [...]
}
```

---

### 10.2 Seller Payouts

```
GET /api/v1/admin/reports/payouts
```

```
POST /api/v1/admin/payouts/process
```

Triggers weekly payout batch to all eligible sellers.

---

### 10.3 Payout Detail

```
GET /api/v1/admin/payouts/:sellerId
```

---

## 11. Platform Configuration

### 11.1 Get Settings

```
GET /api/v1/admin/settings
```

### 11.2 Update Settings

```
PATCH /api/v1/admin/settings
```

**Body**:

```json
{
  "commission": {
    "petSale": 5,
    "productSale": 15,
    "serviceFee": 10
  },
  "moderation": {
    "autoApproveVerifiedBreeders": true,
    "requirePhotoMin": 3,
    "maxListingsPerSeller": 50
  },
  "delivery": {
    "freeDeliveryThreshold": 999,
    "defaultShippingFee": 79
  }
}
```

---

## 12. Audit Log

### 12.1 Get Audit Log

```
GET /api/v1/admin/audit-log
```

**Query Parameters**: `adminId`, `action`, `targetType`, `from`, `to`

**Response**:

```json
{
  "data": [
    {
      "id": "audit_001",
      "adminId": "admin_001",
      "adminName": "Rahul Admin",
      "action": "user.ban",
      "targetType": "user",
      "targetId": "user_456",
      "details": { "reason": "Fraudulent activity" },
      "ip": "103.xx.xx.xx",
      "timestamp": "2026-05-28T10:30:00Z"
    }
  ]
}
```

---

## 13. Error Responses

| Status | Code | Description |
|--------|------|-------------|
| 401 | `ADMIN_AUTH_REQUIRED` | Missing/invalid admin token |
| 403 | `INSUFFICIENT_ROLE` | Role doesn't have permission |
| 403 | `MFA_REQUIRED` | Sensitive action needs MFA |
| 404 | `RESOURCE_NOT_FOUND` | Target entity not found |
| 409 | `ALREADY_PROCESSED` | Action already taken on this item |
