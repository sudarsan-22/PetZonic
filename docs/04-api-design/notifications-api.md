# PetZonic — Notifications API

> **Version**: 1.0.0  
> **Base URL**: `/api/v1/notifications`

---

## 1. Overview

The Notifications API manages push notifications (FCM), in-app notifications, SMS (for critical events), and email notifications. Uses Bull queue for async delivery with retry logic.

---

## 2. Notification Channels

| Channel | Provider | Use Case |
|---------|----------|----------|
| Push (FCM) | Firebase Cloud Messaging | Real-time alerts |
| In-App | Internal | Notification feed in app |
| SMS | MSG91 | OTP, critical alerts |
| Email | AWS SES | Receipts, weekly summaries |

---

## 3. Notification Events

### Buyer Notifications

| Event | Push | In-App | SMS | Email |
|-------|:----:|:------:|:---:|:-----:|
| Order confirmed | ✅ | ✅ | ✅ | ✅ |
| Order shipped | ✅ | ✅ | — | ✅ |
| Order delivered | ✅ | ✅ | — | — |
| Order cancelled | ✅ | ✅ | — | ✅ |
| Refund processed | ✅ | ✅ | — | ✅ |
| Pet listing inquiry reply | ✅ | ✅ | — | — |
| New chat message | ✅ | ✅ | — | — |
| Price drop on wishlist | ✅ | ✅ | — | — |
| New pets matching saved search | ✅ | ✅ | — | — |
| Booking reminder (24h) | ✅ | ✅ | ✅ | — |
| Booking confirmed | ✅ | ✅ | — | ✅ |
| Pet receipt reminder | ✅ | ✅ | — | — |
| Review reminder (7 days) | ✅ | ✅ | — | — |

### Seller Notifications

| Event | Push | In-App | SMS | Email |
|-------|:----:|:------:|:---:|:-----:|
| New order received | ✅ | ✅ | ✅ | ✅ |
| Order cancelled by buyer | ✅ | ✅ | — | ✅ |
| New inquiry on listing | ✅ | ✅ | — | — |
| New chat message | ✅ | ✅ | — | — |
| Listing approved | ✅ | ✅ | — | — |
| Listing rejected | ✅ | ✅ | — | ✅ |
| New review received | ✅ | ✅ | — | — |
| Payout processed | ✅ | ✅ | — | ✅ |
| KYC approved/rejected | ✅ | ✅ | ✅ | ✅ |
| Listing about to expire | ✅ | ✅ | — | — |
| Dispute raised | ✅ | ✅ | ✅ | ✅ |

### Admin Notifications

| Event | Push | In-App | Email |
|-------|:----:|:------:|:-----:|
| New KYC submission | — | ✅ | — |
| Listing pending moderation | — | ✅ | — |
| Dispute raised | — | ✅ | ✅ |
| High-value order (>₹50k) | — | ✅ | ✅ |
| Reported content | — | ✅ | — |
| System alert (errors) | — | — | ✅ |

---

## 4. Endpoints

### 4.1 Get Notifications (In-App Feed)

```
GET /api/v1/notifications
```

**Headers**: `Authorization: Bearer <token>`

**Query Parameters**:

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| page | integer | 1 | Page number |
| limit | integer | 20 | Per page |
| unreadOnly | boolean | false | Only unread |
| type | string | — | Filter: `order`, `chat`, `listing`, `review`, `system` |

**Response** `200 OK`:

```json
{
  "data": [
    {
      "id": "notif_001",
      "type": "order",
      "title": "Order Shipped! 🚚",
      "body": "Your order #PZ-ORD-7890 has been shipped. Track it now!",
      "icon": "package",
      "image": "https://cdn.petzonic.com/products/thumb_rc.jpg",
      "action": {
        "type": "deep_link",
        "url": "/orders/ord_7890"
      },
      "isRead": false,
      "createdAt": "2026-05-28T14:30:00Z"
    },
    {
      "id": "notif_002",
      "type": "chat",
      "title": "New message from Happy Paws",
      "body": "Yes, the puppy is available for visit this weekend!",
      "icon": "chat",
      "action": {
        "type": "deep_link",
        "url": "/chat/room_abc"
      },
      "isRead": true,
      "createdAt": "2026-05-28T12:15:00Z"
    }
  ],
  "unreadCount": 5,
  "pagination": { "page": 1, "limit": 20, "total": 45, "totalPages": 3 }
}
```

---

### 4.2 Get Unread Count

```
GET /api/v1/notifications/unread-count
```

**Response**:

```json
{
  "total": 5,
  "byType": {
    "order": 2,
    "chat": 2,
    "listing": 1,
    "review": 0,
    "system": 0
  }
}
```

---

### 4.3 Mark as Read

```
POST /api/v1/notifications/mark-read
```

**Body**:

```json
{
  "notificationIds": ["notif_001", "notif_002"]
}
```

Or mark all:

```json
{
  "markAll": true
}
```

---

### 4.4 Delete Notification

```
DELETE /api/v1/notifications/:id
```

---

## 5. Notification Preferences

### 5.1 Get Preferences

```
GET /api/v1/notifications/preferences
```

**Response**:

```json
{
  "push": {
    "enabled": true,
    "orders": true,
    "chat": true,
    "promotions": false,
    "priceAlerts": true,
    "bookingReminders": true,
    "newListings": true,
    "reviews": true
  },
  "email": {
    "enabled": true,
    "orderConfirmation": true,
    "shipping": true,
    "promotions": false,
    "weeklySummary": true
  },
  "sms": {
    "enabled": true,
    "otp": true,
    "orderAlerts": true,
    "bookingReminders": true
  },
  "quietHours": {
    "enabled": true,
    "from": "22:00",
    "to": "07:00"
  }
}
```

---

### 5.2 Update Preferences

```
PATCH /api/v1/notifications/preferences
```

**Body** (partial update):

```json
{
  "push": {
    "promotions": false
  },
  "quietHours": {
    "enabled": true,
    "from": "23:00",
    "to": "08:00"
  }
}
```

---

## 6. Device Token Management

### 6.1 Register Device Token

```
POST /api/v1/notifications/devices
```

**Body**:

```json
{
  "token": "fcm_device_token_string",
  "platform": "android",
  "deviceId": "device_unique_id",
  "appVersion": "1.2.0"
}
```

---

### 6.2 Remove Device Token

```
DELETE /api/v1/notifications/devices/:deviceId
```

Called on logout.

---

## 7. Admin: Send Notification

### 7.1 Send to User

```
POST /api/v1/admin/notifications/send
```

**Body**:

```json
{
  "userId": "user_123",
  "title": "Important update about your order",
  "body": "Your refund has been processed.",
  "channels": ["push", "in_app"],
  "action": {
    "type": "deep_link",
    "url": "/orders/ord_123"
  }
}
```

---

### 7.2 Send Broadcast

```
POST /api/v1/admin/notifications/broadcast
```

**Body**:

```json
{
  "segment": "all_buyers",
  "title": "🎉 Mega Sale - Flat 30% off!",
  "body": "Shop pet food & accessories at unbeatable prices.",
  "image": "https://cdn.petzonic.com/banners/sale.jpg",
  "action": {
    "type": "deep_link",
    "url": "/products?sale=mega30"
  },
  "channels": ["push"],
  "scheduledAt": "2026-06-01T09:00:00+05:30"
}
```

**Segments**: `all_users`, `all_buyers`, `all_sellers`, `inactive_7d`, `cart_abandoned`, `city:bangalore`

---

## 8. Internal: Trigger Notification (Service-to-Service)

Used by other backend modules to send notifications:

```typescript
// Internal service call (not HTTP)
notificationService.send({
  userId: 'user_123',
  event: 'ORDER_CONFIRMED',
  data: {
    orderId: 'ord_789',
    orderNumber: 'PZ-ORD-7890',
    amount: 2500
  }
});
```

The notification service resolves the event to a template, checks user preferences, and dispatches via appropriate channels.

---

## 9. Notification Templates

| Event | Title Template | Body Template |
|-------|---------------|--------------|
| `ORDER_CONFIRMED` | Order Confirmed! ✅ | Your order #{{orderNumber}} (₹{{amount}}) is confirmed. |
| `ORDER_SHIPPED` | Order Shipped! 🚚 | Your order #{{orderNumber}} is on its way. Track it now! |
| `ORDER_DELIVERED` | Delivered! 📦 | Your order #{{orderNumber}} has been delivered. |
| `NEW_MESSAGE` | Message from {{senderName}} | {{messagePreview}} |
| `LISTING_APPROVED` | Listing Live! 🎉 | Your {{petType}} listing is now visible to buyers. |
| `PAYOUT_PROCESSED` | Payout Sent! 💰 | ₹{{amount}} has been transferred to your bank account. |
| `BOOKING_REMINDER` | Appointment Tomorrow ⏰ | {{serviceName}} with {{providerName}} at {{time}} |

---

## 10. Rate Limiting & Throttling

| Channel | Limit | Window |
|---------|-------|--------|
| Push per user | 10 | 1 hour |
| SMS per user | 5 | 1 day |
| Email per user | 10 | 1 day |
| Broadcast | 1 | 1 hour |

Quiet hours (default 10 PM - 7 AM): non-critical notifications queued for morning delivery.

---

## 11. Error Responses

| Status | Code | Description |
|--------|------|-------------|
| 400 | `INVALID_TOKEN` | FCM token invalid or expired |
| 404 | `NOTIFICATION_NOT_FOUND` | Notification ID doesn't exist |
| 429 | `RATE_LIMITED` | Too many notifications sent |
| 400 | `INVALID_SEGMENT` | Unknown broadcast segment |
