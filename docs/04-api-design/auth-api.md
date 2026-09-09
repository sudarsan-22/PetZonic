# PetZonic — Auth API

> **Base**: `/api/v1/auth`

---

## POST /auth/otp/send

Send OTP to phone number for login/registration.

**Auth**: None  
**Rate Limit**: 3 requests per hour per phone number

### Request
```json
{
  "phone": "+919876543210"
}
```

### Response (200)
```json
{
  "success": true,
  "data": {
    "message": "OTP sent successfully",
    "expiresIn": 300,
    "isNewUser": true
  }
}
```

### Errors
| Code | Condition |
|------|-----------|
| 400 | Invalid phone format |
| 429 | OTP rate limit exceeded (max 3/hour) |
| 403 | Phone number is banned |

---

## POST /auth/otp/verify

Verify OTP and authenticate user. Creates account if new user.

**Auth**: None

### Request
```json
{
  "phone": "+919876543210",
  "otp": "123456"
}
```

### Response (200)
**Headers:**
```http
Set-Cookie: refreshToken=c2a4f9...; HttpOnly; Secure; SameSite=Lax; Path=/api/v1/auth; Max-Age=604800
```

**Body:**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbG...",
    "expiresIn": 900,
    "user": {
      "id": "uuid",
      "phone": "+919876543210",
      "isNewUser": true,
      "roles": ["BUYER"],
      "profile": null
    }
  }
}
```

### Errors
| Code | Condition |
|------|-----------|
| 400 | Invalid OTP format |
| 401 | Wrong OTP or expired (max 5 attempts) |
| 403 | Account suspended/banned |
| 429 | Too many failed attempts |

---

## POST /auth/register

Register with email and password (alternative to OTP).

**Auth**: None  
**Headers**: `X-Requested-With: XMLHttpRequest` (Anti-CSRF)

### Request
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "Rahul",
  "lastName": "Sharma"
}
```

### Response (201)
**Headers:**
```http
Set-Cookie: refreshToken=c2a4f9...; HttpOnly; Secure; SameSite=Lax; Path=/api/v1/auth; Max-Age=604800
```

**Body:**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbG...",
    "expiresIn": 900,
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "emailVerified": false,
      "roles": ["BUYER"]
    },
    "message": "Verification email sent"
  }
}
```

### Validation Rules
- email: valid email format, unique
- password: min 8 chars, 1 uppercase, 1 number, 1 special character
- firstName: 2-50 characters
- lastName: optional, 2-50 characters

---

## POST /auth/login

Login with email and password.

**Auth**: None  
**Headers**: `X-Requested-With: XMLHttpRequest` (Anti-CSRF)  
**Rate Limit**: 5 attempts per 15 min per IP / identity  
**Anti-Enumeration**: Uses constant-time dummy bcrypt comparison if user does not exist.

> **Admin Isolation Policy**: Customer storefront login strictly rejects accounts with role `ADMIN` (`403 FORBIDDEN: ADMIN_MUST_USE_ADMIN_PORTAL`). Administrative staff must authenticate via the dedicated admin portal endpoint (`/admin/auth/login`).

### Request
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

### Response (200)
**Headers:**
```http
Set-Cookie: refreshToken=c2a4f9...; HttpOnly; Secure; SameSite=Lax; Path=/api/v1/auth; Max-Age=604800
```

**Body:**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbG...",
    "expiresIn": 900,
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "phone": "+919876543210",
      "roles": ["BUYER", "SELLER"],
      "profile": {
        "firstName": "Rahul",
        "lastName": "Sharma",
        "avatarUrl": "https://media.petzonic.com/avatars/...",
        "city": "Bangalore"
      }
    }
  }
}
```

---

## POST /auth/refresh

Get new access token and rotate refresh token using database-atomic test-and-set semantics.

**Auth**: Transmitted via `HttpOnly` Cookie (`refreshToken`)  
**Headers**: `X-Requested-With: XMLHttpRequest` or `X-PetZonic-CSRF: 1` (Dual-Defense CSRF)

### Request
*Empty JSON body or omitted — refresh token automatically parsed from HttpOnly cookie.*

### Response (200)
**Headers:**
```http
Set-Cookie: refreshToken=d5b8e1...(rotated); HttpOnly; Secure; SameSite=Lax; Path=/api/v1/auth; Max-Age=604800
```

**Body:**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbG...(new)",
    "expiresIn": 900
  }
}
```

### Security & Invalidation Semantics
- **Database Atomicity**: `UPDATE refresh_tokens SET revoked_at = NOW() WHERE id = $1 AND revoked_at IS NULL` guarantees exactly 1 winner out of 100+ concurrent requests.
- **Concurrent Losers**: Return `401 TOKEN_ALREADY_ROTATED` without forking new chains.
- **Replay / Theft Detection**: If an already consumed or revoked token is presented, the entire family (`family_id`) is instantly revoked.
- **Multi-Device Isolation**: Replay of Family A does not affect unrelated Family B sessions.

---

## POST /auth/logout

Logout and revoke current refresh session.

**Auth**: Required (Bearer access token or HttpOnly refresh cookie)  
**Headers**: `X-Requested-With: XMLHttpRequest`

### Response (200)
**Headers:**
```http
Set-Cookie: refreshToken=; HttpOnly; Secure; SameSite=Lax; Path=/api/v1/auth; Max-Age=0
```

**Body:**
```json
{
  "success": true,
  "data": { "message": "Logged out successfully" }
}
```

---

## POST /auth/logout-all

Revoke ALL active refresh token sessions across all devices and immediately invalidate in-flight JWTs by incrementing `tokenVersion`.

**Auth**: Required (Bearer access token)  
**Headers**: `X-Requested-With: XMLHttpRequest`

### Response (200)
```json
{
  "success": true,
  "data": { "message": "All sessions revoked successfully" }
}
```

---

## POST /auth/change-password

Change account password, revoke all active sessions, and increment `tokenVersion`.

**Auth**: Required  
**Headers**: `X-Requested-With: XMLHttpRequest`

### Request
```json
{
  "currentPassword": "OldPassword123!",
  "newPassword": "NewSecurePassword456!"
}
```

### Response (200)
```json
{
  "success": true,
  "data": { "message": "Password changed successfully. Please log in again." }
}
```

---

## POST /auth/google

Authenticate via Google OAuth.

**Auth**: None

### Request
```json
{
  "idToken": "google-id-token-from-client-sdk"
}
```

### Response (200)
Same as /auth/otp/verify response.

---

## POST /auth/apple

Authenticate via Apple Sign In.

**Auth**: None

### Request
```json
{
  "identityToken": "apple-identity-token",
  "authorizationCode": "apple-auth-code",
  "fullName": { "givenName": "Rahul", "familyName": "Sharma" }
}
```

---

## POST /auth/forgot-password

Send password reset email.

**Auth**: None

### Request
```json
{
  "email": "user@example.com"
}
```

### Response (200)
```json
{
  "success": true,
  "data": { "message": "Reset link sent if account exists" }
}
```
Note: Always returns success (doesn't reveal if email exists).

---

## POST /auth/reset-password

Reset password using token from email.

**Auth**: None

### Request
```json
{
  "token": "reset-token-from-email",
  "newPassword": "NewSecure456!"
}
```

### Response (200)
```json
{
  "success": true,
  "data": { "message": "Password reset successfully. Please login." }
}
```
