# PetZonic — Orders API

> **Base**: `/api/v1/orders`, `/api/v1/cart`
> **Verified against source**: 2026-09-26 (`petzonic-api/src/modules/orders/`, `cart/`)

> ⚠️ The request/response examples further down were written in May 2026 as a design and
> differ from the code in places (e.g. there is no `productVariantId` or `type` field — items
> are `{ productId | petListingId, quantity }`). The route table and rules in **Implementation
> status** below are measured from source; trust them over the examples.

## Implementation status (2026-09-26)

| Method | Path | Auth | Notes |
|---|---|---|---|
| `GET` / `POST` / `PATCH` / `DELETE` | `/cart`, `/cart/items`, `/cart/items/:id` | Authenticated | Server-backed cart (web syncs its Redux cart to it since 2026-09-18) |
| `POST` | `/orders` | Authenticated | Server computes all totals, tax, shipping, coupon |
| `GET` | `/orders` | Authenticated | Buyer's orders |
| `GET` | `/orders/seller` | Authenticated | Seller's orders |
| `GET` | `/orders/:id` | Authenticated | Detail |
| `POST` | `/orders/:id/ship` | Authenticated (seller) | Courier shipment via logistics factory |
| `PATCH` | `/orders/:id/tracking` | Authenticated (seller) | Manual tracking entry |
| `GET` | `/orders/:id/live-tracking` | Authenticated | Courier checkpoints |
| `GET` | `/orders/:id/shipping-label` | Authenticated (seller) | Label |
| `GET` | `/orders/:id/invoice` | Authenticated (order's buyer or admin) | **New 2026-09-18** — tax invoice as JSON, or printable HTML with `?format=html` or `Accept: text/html` |
| `PATCH` | `/orders/:id/cancel` | Authenticated | Restores stock, reverts pet to `ACTIVE`, decrements coupon usage (one transaction) |
| `POST` | `/orders/:id/return` | Authenticated | Return request |
| `POST` | `/orders/:id/confirm-receipt` | Authenticated (buyer) | Releases escrow if `HELD` |
| `POST` | `/orders/logistics/webhook` | Signature | Shiprocket/Delhivery callbacks |

**Scheduled rules (since 2026-09-22, `petzonic-maintenance-queue`)**
- **Abandoned orders**: every 15 minutes, `PENDING_PAYMENT` orders older than **30 minutes** are
  cancelled; stock and pet listings return to circulation (`expireAbandonedOrders()`).
- **Escrow auto-release**: hourly, pet orders delivered 7+ days ago with no dispute move from
  `HELD` to `RELEASED` (`autoReleaseExpiredEscrows()`).
- Pet purchase stays an atomic guarded transition: listing `ACTIVE → PAUSED` at order creation.

---

---

## Cart Endpoints

### GET /cart
Get current user's cart.

**Auth**: Required

**Response (200)**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "items": [
      {
        "id": "cart-item-uuid",
        "product": {
          "id": "uuid",
          "name": "Premium Dog Food 5kg",
          "imageUrl": "..."
        },
        "variant": {
          "id": "uuid",
          "name": "Chicken Flavor - 5kg",
          "sku": "PDF-CHK-5KG",
          "price": 1299,
          "stockQuantity": 45
        },
        "quantity": 2,
        "subtotal": 2598
      }
    ],
    "summary": {
      "itemCount": 2,
      "subtotal": 2598,
      "estimatedShipping": 49,
      "estimatedTotal": 2647
    }
  }
}
```

### POST /cart/items
Add item to cart.

**Request**:
```json
{ "productVariantId": "uuid", "quantity": 1 }
```

### PATCH /cart/items/:id
Update quantity.

**Request**: `{ "quantity": 3 }`

### DELETE /cart/items/:id
Remove item from cart.

---

## Order Endpoints

### POST /orders

Place an order (checkout).

**Auth**: Required

**Request**:
```json
{
  "type": "PRODUCT",
  "addressId": "uuid",
  "items": [
    { "productVariantId": "uuid", "quantity": 2 }
  ],
  "couponCode": "FLAT100",
  "deliverySlot": "STANDARD",
  "paymentMethod": "UPI",
  "notes": "Leave at door"
}
```

For pet purchase:
```json
{
  "type": "PET",
  "petListingId": "uuid",
  "paymentMethod": "UPI"
}
```

**Response (201)**:
```json
{
  "success": true,
  "data": {
    "orderId": "uuid",
    "orderNumber": "PZ-20260528-4821",
    "total": 2547,
    "razorpayOrderId": "order_N3b2cDeFgH",
    "status": "PENDING_PAYMENT"
  }
}
```

### GET /orders

List user's orders.

**Query**: `?status=DELIVERED&page=1&limit=10`

### GET /orders/:id

Get order details with tracking.

**Response includes**: items, payment info, tracking number/URL, timeline of status changes.

### PATCH /orders/:id/cancel

Cancel an order.

**Request**: `{ "reason": "Changed my mind" }`

**Rules**: Only cancellable in CONFIRMED or PROCESSING status.

### POST /orders/:id/return

Initiate a return.

**Request**:
```json
{
  "itemId": "order-item-uuid",
  "reason": "DEFECTIVE",
  "description": "Product arrived damaged",
  "photoUrls": ["..."]
}
```

**Rules**: Only within 7 days of delivery.

### POST /orders/:id/confirm-receipt

Buyer confirms pet receipt (releases escrow).

**Auth**: Required (buyer only)

**Request**: `{ "confirmed": true }`

**Effect**: Releases escrow payment to seller (minus commission).
