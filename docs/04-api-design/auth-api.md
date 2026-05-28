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
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbG...",
    "refreshToken": "eyJhbG...",
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
| 401 | Wrong OTP or expired |
| 403 | Account suspended/banned |
| 429 | Too many failed attempts (5 max) |

---

## POST /auth/register

Register with email and password (alternative to OTP).

**Auth**: None

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
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbG...",
    "refreshToken": "eyJhbG...",
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
**Rate Limit**: 5 attempts per 15 min per IP

### Request
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

### Response (200)
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbG...",
    "refreshToken": "eyJhbG...",
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

Get new access token using refresh token.

**Auth**: None (refresh token in body)

### Request
```json
{
  "refreshToken": "eyJhbG..."
}
```

### Response (200)
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbG...(new)",
    "refreshToken": "eyJhbG...(new, rotated)",
    "expiresIn": 900
  }
}
```

### Errors
| Code | Condition |
|------|-----------|
| 401 | Invalid or expired refresh token |
| 401 | Refresh token reuse detected (all sessions revoked) |

---

## POST /auth/logout

Logout and revoke current session.

**Auth**: Required

### Request
```json
{
  "refreshToken": "eyJhbG...",
  "deviceId": "device-uuid"
}
```

### Response (200)
```json
{
  "success": true,
  "data": { "message": "Logged out successfully" }
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
