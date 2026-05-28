# PetZonic — Payments API

> **Base**: `/api/v1/payments`

---

## POST /payments/create-order

Create a Razorpay payment order for checkout.

**Auth**: Required

**Request**:
```json
{
  "orderId": "uuid",
  "amount": 2547
}
```

**Response (200)**:
```json
{
  "success": true,
  "data": {
    "razorpayOrderId": "order_N3b2cDeFgH",
    "amount": 254700,
    "currency": "INR",
    "key": "rzp_live_XXXXXXXXXX",
    "prefill": {
      "name": "Rahul Sharma",
      "email": "rahul@example.com",
      "contact": "+919876543210"
    }
  }
}
```

Note: Amount in paise (×100) for Razorpay.

---

## POST /payments/verify

Verify payment after Razorpay callback (called by client after payment success).

**Auth**: Required

**Request**:
```json
{
  "razorpayOrderId": "order_N3b2cDeFgH",
  "razorpayPaymentId": "pay_AbCdEfGhIj",
  "razorpaySignature": "hmac_sha256_signature"
}
```

**Response (200)**:
```json
{
  "success": true,
  "data": {
    "orderId": "uuid",
    "orderNumber": "PZ-20260528-4821",
    "status": "CONFIRMED",
    "message": "Payment successful! Order confirmed."
  }
}
```

**Verification Process**:
1. Validate HMAC signature (razorpay_order_id + "|" + razorpay_payment_id)
2. Verify payment status via Razorpay API
3. Update order status to CONFIRMED
4. For pet orders: create escrow hold
5. Deduct product stock
6. Trigger confirmation notifications

---

## POST /payments/webhook

Razorpay webhook endpoint (server-to-server).

**Auth**: Webhook signature verification (X-Razorpay-Signature header)

**Events handled**:
- `payment.captured` — Confirm order
- `payment.failed` — Mark payment failed
- `refund.processed` — Mark refund complete
- `transfer.processed` — Confirm seller payout

---

## GET /payments/history

Get user's payment history.

**Auth**: Required

**Query**: `?page=1&limit=20`

**Response**: List of payments with order reference, amount, status, method, date.

---

## POST /payments/refund/:orderId

Initiate refund for an order.

**Auth**: Required (buyer for returns, ADMIN for disputes)

**Request**:
```json
{
  "reason": "Product defective",
  "amount": 1299,
  "type": "PARTIAL"
}
```

type: `FULL` or `PARTIAL`

---

## Escrow Flow (Pet Purchases)

```
1. Buyer pays → Payment captured → Escrow HELD
2. Seller accepts order → Meetup/delivery arranged
3a. Buyer confirms receipt → Escrow RELEASED → Payout to seller
3b. Buyer raises dispute → Escrow DISPUTED → Admin resolves
3c. 7 days pass, no dispute → Escrow auto-RELEASED
```

---

## GET /seller/earnings

Seller's earnings summary.

**Auth**: Required (SELLER role)

**Response**:
```json
{
  "success": true,
  "data": {
    "totalEarned": 125000,
    "pendingSettlement": 15000,
    "lastPayout": 35000,
    "lastPayoutDate": "2026-05-21",
    "commission": {
      "rate": 10,
      "totalPaid": 14000
    },
    "thisMonth": {
      "sales": 8,
      "revenue": 45000,
      "commission": 4500,
      "netEarnings": 40500
    }
  }
}
```

---

## GET /seller/payouts

Seller's payout history.

**Auth**: Required (SELLER role)

**Response**: List of payouts with amount, commission, net, status, date.

---

## POST /seller/bank-account

Add/update seller's bank account for payouts.

**Auth**: Required (SELLER role)

**Request**:
```json
{
  "accountHolderName": "Rahul Sharma",
  "accountNumber": "1234567890",
  "ifscCode": "SBIN0001234",
  "bankName": "State Bank of India"
}
```
