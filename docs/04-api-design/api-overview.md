# PetZonic — API Overview

> **Version**: 1.0.0  
> **Date**: May 28, 2026  
> **Base URL**: `https://api.petzonic.com/api/v1`

---

## 1. API Conventions

### Base URLs
| Environment | URL |
|-------------|-----|
| Production | `https://api.petzonic.com/api/v1` |
| Staging | `https://staging-api.petzonic.com/api/v1` |
| Local | `http://localhost:3000/api/v1` |

### Versioning
- URL-based: `/api/v1/...`
- Major version changes = new URL path
- Minor/patch changes = backward compatible

### Request Format
- Content-Type: `application/json`
- File uploads: `multipart/form-data`
- UTF-8 encoding

### Response Format
All responses follow this structure:
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation successful",
  "meta": {
    "timestamp": "2026-05-28T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      { "field": "price", "message": "Price must be at least 100" }
    ]
  },
  "meta": {
    "timestamp": "2026-05-28T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

### Error Codes

| HTTP Status | Error Code | Description |
|:-----------:|-----------|-------------|
| 400 | VALIDATION_ERROR | Invalid input data |
| 400 | BAD_REQUEST | Malformed request |
| 401 | UNAUTHORIZED | Missing or invalid token |
| 401 | TOKEN_EXPIRED | Access token expired |
| 403 | FORBIDDEN | Insufficient permissions |
| 404 | NOT_FOUND | Resource doesn't exist |
| 409 | CONFLICT | Resource already exists |
| 422 | UNPROCESSABLE | Valid format but business rule violation |
| 429 | RATE_LIMITED | Too many requests |
| 500 | INTERNAL_ERROR | Server error |

---

## 2. Authentication

### Headers
```
Authorization: Bearer <access_token>
```

### Token Refresh
When access token expires (401 + TOKEN_EXPIRED), call:
```
POST /auth/refresh
Body: { "refreshToken": "<refresh_token>" }
```

### Public Endpoints (No auth required)
- `POST /auth/otp/send`
- `POST /auth/otp/verify`
- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh`
- `GET /pets` (browse listings)
- `GET /pets/:id` (view listing detail)
- `GET /products` (browse products)
- `GET /products/:id` (view product)
- `GET /species` (list species)
- `GET /breeds` (list breeds)

---

## 3. Pagination

All list endpoints support cursor-based or offset pagination:

### Offset Pagination (default for most endpoints)
```
GET /products?page=1&limit=20&sortBy=createdAt&sortOrder=desc
```

Response includes:
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  }
}
```

### Cursor Pagination (for chat messages, notifications)
```
GET /chat/rooms/:id/messages?cursor=<message_id>&limit=50&direction=before
```

---

## 4. Filtering & Sorting

### Filter Syntax
```
GET /pets?species=dog&breed=golden-retriever&minPrice=5000&maxPrice=50000&city=bangalore&gender=male
```

### Sort Syntax
```
GET /pets?sortBy=price&sortOrder=asc
GET /products?sortBy=rating&sortOrder=desc
```

### Available Sort Fields
| Endpoint | Sort Fields |
|----------|-------------|
| /pets | price, createdAt, distance |
| /products | price, rating, createdAt, name |
| /orders | placedAt, total |
| /reviews | createdAt, rating, helpfulCount |

---

## 5. Rate Limits

| Category | Limit | Window | Header |
|----------|-------|--------|--------|
| Public | 100 req | 1 min / IP | X-RateLimit-Limit |
| Authenticated | 1000 req | 1 min / user | X-RateLimit-Remaining |
| OTP send | 3 req | 1 hour / phone | X-RateLimit-Reset |
| File upload | 20 req | 1 min / user | |
| Search | 60 req | 1 min / user | |

Response headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 995
X-RateLimit-Reset: 1716883260
```

---

## 6. File Upload

### Direct Upload (Pre-signed URL)
```
1. POST /media/upload-url → Returns pre-signed S3 URL
2. PUT <pre-signed-url> (upload file directly to S3)
3. Use returned URL in subsequent API calls
```

### Multipart Upload (via API)
```
POST /media/upload
Content-Type: multipart/form-data
Body: file (binary), type (pet|product|avatar|document)
```

---

## 7. WebSocket (Chat)

### Connection
```
ws://api.petzonic.com/chat?token=<access_token>
```

### Events (Client → Server)
| Event | Payload | Description |
|-------|---------|-------------|
| `join_room` | `{ roomId }` | Join a chat room |
| `leave_room` | `{ roomId }` | Leave a chat room |
| `send_message` | `{ roomId, content, type }` | Send message |
| `typing` | `{ roomId }` | Typing indicator |
| `mark_read` | `{ roomId, messageId }` | Mark messages as read |

### Events (Server → Client)
| Event | Payload | Description |
|-------|---------|-------------|
| `new_message` | `{ message }` | New message received |
| `user_typing` | `{ roomId, userId }` | Other user is typing |
| `message_read` | `{ roomId, messageId }` | Message read receipt |
| `user_online` | `{ userId }` | User came online |
| `user_offline` | `{ userId }` | User went offline |

---

## 8. API Endpoint Index

### Auth (`/auth`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /auth/otp/send | Send OTP to phone |
| POST | /auth/otp/verify | Verify OTP and login |
| POST | /auth/register | Register with email/password |
| POST | /auth/login | Login with email/password |
| POST | /auth/refresh | Refresh access token |
| POST | /auth/logout | Logout (revoke tokens) |
| POST | /auth/forgot-password | Send password reset email |
| POST | /auth/reset-password | Reset password with token |
| POST | /auth/google | Google OAuth login |
| POST | /auth/apple | Apple OAuth login |

### Users (`/users`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /users/me | Get current user profile |
| PATCH | /users/me | Update profile |
| POST | /users/me/avatar | Upload avatar |
| GET | /users/me/roles | Get user roles |
| POST | /users/me/roles | Add role (become seller, etc.) |
| GET | /users/me/devices | List devices |
| DELETE | /users/me/devices/:id | Remove device |
| POST | /users/me/kyc | Submit KYC documents |
| GET | /users/me/kyc/status | Check KYC status |
| DELETE | /users/me | Deactivate account |
| GET | /users/:id/public | Get public profile |

### Pets (`/pets`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /pets | List/search pet listings |
| GET | /pets/:id | Get listing detail |
| POST | /pets | Create pet listing |
| PATCH | /pets/:id | Update listing |
| DELETE | /pets/:id | Delete listing |
| PATCH | /pets/:id/status | Change status (pause/resume/sold) |
| POST | /pets/:id/boost | Boost listing |
| GET | /pets/:id/similar | Get similar listings |
| GET | /pets/my-listings | Seller's own listings |
| POST | /pets/:id/report | Report listing |

### Products (`/products`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /products | List/search products |
| GET | /products/:id | Get product detail |
| GET | /products/categories | List categories |
| GET | /products/:id/reviews | Get product reviews |
| GET | /products/deals | Get active deals |

### Cart & Orders (`/cart`, `/orders`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /cart | Get current cart |
| POST | /cart/items | Add item to cart |
| PATCH | /cart/items/:id | Update quantity |
| DELETE | /cart/items/:id | Remove from cart |
| POST | /orders | Place order (checkout) |
| GET | /orders | List user's orders |
| GET | /orders/:id | Get order detail |
| PATCH | /orders/:id/cancel | Cancel order |
| POST | /orders/:id/return | Initiate return |
| POST | /orders/:id/confirm-receipt | Confirm pet receipt |

### Payments (`/payments`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /payments/create-order | Create Razorpay order |
| POST | /payments/verify | Verify payment (webhook) |
| GET | /payments/history | Payment history |
| POST | /payments/refund/:id | Initiate refund |

### Chat (`/chat`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /chat/rooms | List chat rooms |
| POST | /chat/rooms | Create/get chat room |
| GET | /chat/rooms/:id/messages | Get messages (paginated) |
| POST | /chat/rooms/:id/messages | Send message (REST fallback) |
| POST | /chat/rooms/:id/block | Block user in chat |

### Services (`/services`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /services/providers | List service providers |
| GET | /services/providers/:id | Get provider detail |
| GET | /services/providers/:id/slots | Get available slots |
| POST | /services/bookings | Create booking |
| GET | /services/bookings | List user's bookings |
| PATCH | /services/bookings/:id | Update booking (cancel/reschedule) |

### Reviews (`/reviews`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /reviews | Create review |
| PATCH | /reviews/:id | Edit review |
| DELETE | /reviews/:id | Delete own review |
| POST | /reviews/:id/helpful | Mark as helpful |
| POST | /reviews/:id/respond | Seller responds |

### Notifications (`/notifications`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /notifications | List notifications |
| PATCH | /notifications/:id/read | Mark as read |
| PATCH | /notifications/read-all | Mark all as read |
| GET | /notifications/unread-count | Get unread count |
| PATCH | /notifications/preferences | Update preferences |

### Search (`/search`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /search | Global search (pets + products) |
| GET | /search/suggestions | Autocomplete suggestions |
| GET | /search/trending | Trending searches |

### Admin (`/admin`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /admin/dashboard | Dashboard metrics |
| GET | /admin/users | List all users |
| PATCH | /admin/users/:id/status | Suspend/ban user |
| GET | /admin/moderation/listings | Pending listings queue |
| PATCH | /admin/moderation/listings/:id | Approve/reject listing |
| GET | /admin/orders | All orders |
| POST | /admin/orders/:id/refund | Force refund |
| GET | /admin/revenue | Revenue reports |
| POST | /admin/products | Add product (store) |
| PATCH | /admin/products/:id | Edit product |
| POST | /admin/promotions | Create coupon/deal |
| GET | /admin/disputes | Dispute queue |
| PATCH | /admin/disputes/:id | Resolve dispute |

### Seller (`/seller`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /seller/dashboard | Seller metrics |
| GET | /seller/orders | Orders for seller |
| PATCH | /seller/orders/:id/accept | Accept pet order |
| GET | /seller/earnings | Earnings summary |
| GET | /seller/payouts | Payout history |
| POST | /seller/bank-account | Add bank details |

### Taxonomy (`/species`, `/breeds`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /species | List all species |
| GET | /species/:id/breeds | Breeds for species |
| GET | /breeds/popular | Popular breeds |

### Addresses (`/addresses`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /addresses | List user addresses |
| POST | /addresses | Add address |
| PATCH | /addresses/:id | Update address |
| DELETE | /addresses/:id | Delete address |
| PATCH | /addresses/:id/default | Set as default |

### Wishlist (`/wishlist`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /wishlist | Get wishlist items |
| POST | /wishlist | Add to wishlist (pet or product) |
| DELETE | /wishlist/:id | Remove from wishlist |

---

## 9. API Documentation (Swagger)

Auto-generated Swagger UI available at:
- Production: `https://api.petzonic.com/docs`
- Staging: `https://staging-api.petzonic.com/docs`
- Local: `http://localhost:3000/docs`

Generated from NestJS decorators using `@nestjs/swagger`.
