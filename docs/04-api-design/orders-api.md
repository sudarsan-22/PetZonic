# PetZonic — Orders API

> **Base**: `/api/v1/orders`, `/api/v1/cart`

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
