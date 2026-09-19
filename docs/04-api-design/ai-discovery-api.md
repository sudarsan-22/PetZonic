# PetZonic — AI Shopping & Discovery API

> **Base URL**: `/api/v1/discovery` — also reachable at `/api/v1/chat/discovery`  
> **Version**: 1.0.0  
> **Status**: Implemented and working in development. Never deployed to production.  
> **Verified against source**: 2026-09-20 (`petzonic-api/src/app.ts`, `src/modules/ai-discovery/`)

> ⚠️ **Path corrections.** This document was written against a `/api/v1/ai-discovery` base
> that was never used. Corrections verified 2026-09-20:
>
> | Documented | Actual |
> |---|---|
> | Base `/api/v1/ai-discovery` | `/api/v1/discovery` (and `/api/v1/chat/discovery`) |
> | `POST /ai-discovery/reset` | `DELETE /session/:id` |
> | `GET /ai-discovery/session` | `GET /session/:id` |
> | `GET /ai-discovery/health` | Does not exist — use `GET /metrics` |
>
> Only `POST /chat` matches as documented.

---

## 1. Overview

The AI Shopping & Product Discovery Engine provides a conversational, natural-language interface for discovering pet supplies, food, grooming items, toys, and healthcare products across the PetZonic e-commerce catalog.

### Core Architecture
- **Hybrid Intent Extraction**:
  - **Fast-Path Rule Engine**: Sub-millisecond regex & heuristic matching for common high-confidence patterns (e.g., "dog food under 1500", "cat toys", "shampoo").
  - **LLM Provider Fallback**: Deploys local Ollama container (`qwen2.5:3b` / `llama3.2:3b`) or Google Gemini (`gemini-2.5-flash`) for multi-turn disambiguation, Hindi/English (Hinglish) queries, negation ("actually no treats"), and conversational shifts.
- **Canary Rollout Mechanism**: Controlled by `AI_DISCOVERY_ROLLOUT_PERCENTAGE` (default 15%), hashing user/session IDs to ensure deterministic cohort assignment.
- **Session Persistence**: Stored in Redis with sliding-window TTL (30 minutes), maintaining conversation turns, active category/species filters, price constraints, and sort preferences.

---

## 2. Endpoints

### POST /ai-discovery/chat

Send a conversational query to discover products and update the active shopping session.

**Auth**: Optional (Supports both guest sessions via `sessionId` and authenticated users)  
**Rate Limit**: 60 requests per minute per IP / User  

#### Request
```json
{
  "message": "Show me healthy dog food under ₹1500",
  "sessionId": "sess_89abc471-ef62-4211-9f21"
}
```

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `message` | string | Yes | Customer's natural language shopping query (1–500 characters) |
| `sessionId` | string | No | Existing session ID for multi-turn continuity. If omitted, a new UUID is generated. |

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "message": "Here are nutritious dog food options under ₹1,500 that your dog will love:",
    "sessionId": "sess_89abc471-ef62-4211-9f21",
    "turnCount": 1,
    "activeFilters": {
      "species": "DOG",
      "categorySlug": "dog-food",
      "maxPrice": 1500,
      "sortBy": "rating"
    },
    "products": [
      {
        "id": "prod_123e4567-e89b-12d3-a456-426614174000",
        "name": "Pedigree Adult Dry Dog Food - Chicken & Vegetables 3kg",
        "slug": "pedigree-adult-dry-dog-food-chicken-vegetables-3kg",
        "brand": "Pedigree",
        "price": 899.00,
        "originalPrice": 1050.00,
        "rating": 4.6,
        "reviewCount": 184,
        "inStock": true,
        "thumbnailUrl": "https://media.petzonic.com/products/pedigree-chicken.jpg",
        "category": {
          "id": "cat_dog_food",
          "name": "Dog Food",
          "slug": "dog-food"
        }
      }
    ],
    "totalMatches": 14
  }
}
```

#### Errors
| Code | Reason |
|:----:|--------|
| `400` | Malformed JSON or empty message |
| `429` | Rate limit exceeded |
| `503` | AI provider unavailable (service degrades gracefully with helpful suggestion) |

---

### POST /ai-discovery/reset

Reset the conversational context, clear active filter constraints, and start a new shopping session.

**Auth**: Optional  

#### Request
```json
{
  "sessionId": "sess_89abc471-ef62-4211-9f21"
}
```

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "message": "Shopping session reset successfully",
    "sessionId": "sess_89abc471-ef62-4211-9f21"
  }
}
```

---

### GET /ai-discovery/session

Inspect the current session's active filters, query history, and turn count.

**Auth**: Optional  

#### Query Parameters
- `sessionId`: string (Required)

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "sessionId": "sess_89abc471-ef62-4211-9f21",
    "turnCount": 3,
    "activeFilters": {
      "species": "CAT",
      "categorySlug": "cat-food",
      "minPrice": 500,
      "maxPrice": 2000
    },
    "history": [
      { "role": "user", "content": "Looking for cat food" },
      { "role": "assistant", "content": "I found 8 cat foods for you." },
      { "role": "user", "content": "Only premium ones above 500" }
    ],
    "expiresInSeconds": 1640
  }
}
```

---

### GET /ai-discovery/health

Verify the operational status, current active provider (`ollama` / `gemini` / `mock`), and round-trip inference latency.

**Auth**: None (Public health monitoring)  

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "status": "healthy",
    "provider": "ollama",
    "model": "qwen2.5:3b",
    "latencyMs": 42,
    "rolloutPercentage": 15,
    "timestamp": "2026-09-10T00:00:00.000Z"
  }
}
```

---

## 3. Multi-Turn Conversational Examples

### Scenario 1: Refinement & Narrowing
1. User: *"Show me dog food"*  
   → Filters applied: `{ species: 'DOG', categorySlug: 'dog-food' }`
2. User: *"Only under 800"*  
   → Filters updated: `{ species: 'DOG', categorySlug: 'dog-food', maxPrice: 800 }`

### Scenario 2: Category Switching ("Actually...")
1. User: *"Show me chew toys"*  
   → Filters applied: `{ categorySlug: 'toys' }`
2. User: *"Actually, I want orthopedic beds for large dogs"*  
   → Filters reset & updated: `{ species: 'DOG', categorySlug: 'beds-furniture', search: 'orthopedic' }`

### Scenario 3: Zero-Result Recovery
1. User: *"Show dog food under ₹5"*  
   → Zero items match price constraint. Assistant returns: *"We couldn't find dog food under ₹5. Most quality options start at ₹250. Would you like to see affordable options?"*
