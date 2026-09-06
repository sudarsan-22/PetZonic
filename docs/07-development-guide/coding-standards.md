# PetZonic — Coding Standards & Conventions

> **Version**: 1.0.0  
> **Date**: May 28, 2026

---

## 1. General Principles

- **Consistency over preference** — Follow project conventions even if you personally prefer another style.
- **Readability over cleverness** — Code is read 10x more than written.
- **Explicit over implicit** — Name things clearly, avoid abbreviations.
- **Fail fast** — Validate at boundaries, throw early.
- **No dead code** — Remove unused imports, variables, and functions.

---

## 2. TypeScript (Backend + Web)

### 2.1 Formatting

| Rule | Value |
|------|-------|
| Indentation | 2 spaces |
| Semicolons | Yes (always) |
| Quotes | Single quotes `'` |
| Trailing comma | ES5 (last item in multiline) |
| Max line length | 100 characters (soft limit) |
| End of file | Single newline |

**Enforced by**: ESLint + Prettier (auto-format on save)

### 2.2 Naming Conventions

| Entity | Convention | Example |
|--------|-----------|---------|
| Variables, functions | camelCase | `getUserById`, `isActive` |
| Classes, interfaces, types | PascalCase | `UserService`, `CreatePetDto` |
| Enums | PascalCase (keys UPPER_SNAKE) | `OrderStatus.PENDING` |
| Constants | UPPER_SNAKE_CASE | `MAX_UPLOAD_SIZE`, `JWT_SECRET` |
| Files (general) | kebab-case | `user.service.ts`, `create-pet.dto.ts` |
| Directories | kebab-case | `pet-listings/`, `auth-guards/` |
| Database tables | snake_case (plural) | `pet_listings`, `order_items` |
| Database columns | snake_case | `created_at`, `seller_id` |
| API endpoints | kebab-case (plural nouns) | `/api/v1/pet-listings` |
| Environment variables | UPPER_SNAKE_CASE | `DATABASE_URL`, `REDIS_HOST` |

### 2.3 TypeScript Rules

```typescript
// ✅ Use explicit return types on public methods
async findById(id: string): Promise<User> { }

// ✅ Use interfaces for DTOs and data shapes
interface CreatePetDto {
  species: string;
  breed: string;
  price: number;
}

// ✅ Use enums for fixed sets
enum OrderStatus {
  PENDING = 'PENDING',
  CONFIRMED = 'CONFIRMED',
  SHIPPED = 'SHIPPED',
}

// ✅ Avoid `any` — use `unknown` if type is truly unknown
function parseResponse(data: unknown): UserResponse { }

// ❌ Never use `any`
function bad(data: any) { } // NO

// ✅ Use optional chaining and nullish coalescing
const city = user?.address?.city ?? 'Unknown';

// ✅ Destructure where it improves readability
const { name, email, phone } = createUserDto;

// ✅ Use template literals over concatenation
const message = `Welcome, ${user.name}!`;
```

### 2.4 Import Order

```typescript
// 1. Node.js built-in modules
import { join } from 'path';

// 2. Third-party packages
import express, { Request, Response, NextFunction } from 'express';
import { z } from 'zod';
import { prisma } from '../../lib/prisma.js';

// 3. Internal shared modules
import { AppError } from '../../lib/errors.js';
import { sendSuccess } from '../../lib/response.js';

// 4. Feature module relative imports
import { petsService } from './pets.service.js';
import { createPetListingSchema } from './pets.schema.js';
```

---

## 3. Express 5 (Backend) Conventions

### 3.1 4-Tier Module Structure

Every domain module in `src/modules/` follows a strict 4-tier layer pattern:

```
src/
├── app.ts                        # Express 5 app setup, middleware, root router
├── server.ts                     # HTTP listener, Socket.io initialization
├── config/                       # Typed environment configuration
│   └── env.ts
├── lib/                          # Core shared infrastructure
│   ├── prisma.ts                 # Prisma ORM singleton client
│   ├── redis.ts                  # Redis connection & caching helpers
│   ├── errors.ts                 # AppError and custom error hierarchy
│   ├── response.ts               # Standardized sendSuccess, sendError helpers
│   ├── logger.ts                 # Structured Pino logger
│   └── validation.ts             # Reusable Zod schema primitives
├── middleware/                   # Express middleware
│   ├── auth.ts                   # JWT authentication & requireRole guards
│   ├── error-handler.ts          # Centralized error formatting middleware
│   ├── rate-limit.ts             # Redis-backed rate limiters
│   └── request-logger.ts         # HTTP request logging
├── modules/                      # 19 self-contained domain modules
│   ├── pets/
│   │   ├── pets.router.ts        # HTTP route definitions & route middleware
│   │   ├── pets.controller.ts    # Request parsing, DTO validation, HTTP response
│   │   ├── pets.service.ts       # Core business logic & cross-module orchestration
│   │   ├── pets.repository.ts    # Prisma database access layer
│   │   ├── pets.schema.ts        # Zod request/response validation schemas
│   │   ├── pet-ai.provider.ts    # Domain-specific integrations (e.g. Gemini AI)
│   │   └── pets.router.test.ts   # Vitest + Supertest integration tests
│   ├── auth/
│   ├── orders/
│   ├── products/
│   └── ...
└── prisma/
    ├── schema.prisma             # Unified PostgreSQL schema (58 models)
    ├── migrations/
    └── seed.ts
```

### 3.2 Router Rules (`*.router.ts`)

```typescript
import { Router } from "express";
import { authenticate, requireRole } from "../../middleware/auth.js";
import { petsController } from "./pets.controller.js";

export const petsRouter = Router();

// Public routes
petsRouter.get("/", petsController.listPets);
petsRouter.get("/:id", petsController.getPetById);

// Protected routes
petsRouter.post("/", authenticate, requireRole(["SELLER", "BREEDER", "ADMIN"]), petsController.createPetListing);
petsRouter.patch("/:id", authenticate, petsController.updatePetListing);
petsRouter.delete("/:id", authenticate, petsController.deletePetListing);
```

### 3.3 Controller Rules (`*.controller.ts`)

```typescript
import { Request, Response, NextFunction } from "express";
import { sendSuccess } from "../../lib/response.js";
import { createPetListingSchema, listPetsQuerySchema } from "./pets.schema.js";
import { petsService } from "./pets.service.js";

export class PetsController {
  // ✅ Parse and validate query/body with Zod
  // ✅ Delegate business logic entirely to the service layer
  // ✅ Format response with sendSuccess
  async createPetListing(req: Request, res: Response, next: NextFunction) {
    try {
      const input = createPetListingSchema.parse(req.body);
      const pet = await petsService.createPet(req.user!.id, input);
      return sendSuccess(res, pet, 201);
    } catch (err) {
      next(err);
    }
  }

  async listPets(req: Request, res: Response, next: NextFunction) {
    try {
      const query = listPetsQuerySchema.parse(req.query);
      const result = await petsService.listPets(query);
      return sendSuccess(res, result.items, 200, result.pagination);
    } catch (err) {
      next(err);
    }
  }
}

export const petsController = new PetsController();
```

### 3.4 Service Rules (`*.service.ts`)

```typescript
import { AppError } from "../../lib/errors.js";
import { petsRepository } from "./pets.repository.js";
import type { CreatePetListingInput, ListPetsQuery } from "./pets.schema.js";

export class PetsService {
  // ✅ Business logic and authorization checks live here
  async getPetById(id: string) {
    const pet = await petsRepository.findById(id);
    if (!pet) {
      throw new AppError("Pet listing not found", 404);
    }
    return pet;
  }

  // ✅ Multi-table atomic operations use prisma.$transaction
  async createPet(sellerId: string, input: CreatePetListingInput) {
    return petsRepository.create(sellerId, input);
  }
}

export const petsService = new PetsService();
```

### 3.5 Schema Validation with Zod (`*.schema.ts`)

```typescript
import { z } from "zod";
import { httpUrlSchema } from "../../lib/validation.js";

export const createPetListingSchema = z.object({
  speciesId: z.string().uuid(),
  breedId: z.string().uuid(),
  title: z.string().min(5).max(200),
  description: z.string().min(20),
  gender: z.enum(["MALE", "FEMALE"]),
  ageMonths: z.number().min(0).max(300),
  price: z.number().min(0).max(10_000_000),
  priceType: z.enum(["FIXED", "NEGOTIABLE"]).default("FIXED"),
  city: z.string().min(1),
  state: z.string().min(1),
  isVaccinated: z.boolean().default(false),
  isNeutered: z.boolean().default(false),
  images: z.array(httpUrlSchema).min(1).max(10),
});

export type CreatePetListingInput = z.infer<typeof createPetListingSchema>;
```

---

## 4. Flutter (Mobile) Conventions

### 4.1 Project Structure

```
lib/
├── main.dart
├── app.dart
├── core/
│   ├── constants/
│   ├── errors/
│   ├── network/
│   │   ├── api_client.dart
│   │   └── interceptors/
│   ├── router/
│   ├── theme/
│   └── utils/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── models/
│   │   │   ├── repositories/
│   │   │   └── datasources/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   └── usecases/
│   │   └── presentation/
│   │       ├── providers/  (or bloc/)
│   │       ├── screens/
│   │       └── widgets/
│   ├── pets/
│   ├── products/
│   └── ...
├── shared/
│   ├── widgets/
│   ├── models/
│   └── providers/
└── l10n/
    ├── app_en.arb
    └── app_hi.arb
```

### 4.2 Dart Naming

| Entity | Convention | Example |
|--------|-----------|---------|
| Files | snake_case | `pet_detail_screen.dart` |
| Classes | PascalCase | `PetDetailScreen` |
| Variables/functions | camelCase | `getPetById()` |
| Constants | camelCase with `k` prefix | `kDefaultPadding` |
| Private members | `_` prefix | `_isLoading` |
| Widgets | PascalCase | `PetCard`, `PriceTag` |
| Providers | camelCase + `Provider` suffix | `petListProvider` |

### 4.3 Widget Rules

```dart
// ✅ Extract widgets into separate files when > 50 lines
// ✅ Use const constructors where possible
class PetCard extends StatelessWidget {
  const PetCard({super.key, required this.pet});
  
  final Pet pet;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        children: [
          _buildImage(),
          _buildInfo(),
        ],
      ),
    );
  }

  // ✅ Use private methods for sub-sections
  Widget _buildImage() => ...;
  Widget _buildInfo() => ...;
}

// ❌ Don't put business logic in widgets
// ❌ Don't use setState for complex state — use Riverpod/Bloc
```

### 4.4 State Management (Riverpod)

```dart
// ✅ One provider per concern
final petListProvider = FutureProvider.autoDispose
    .family<List<Pet>, PetFilter>((ref, filter) async {
  final repository = ref.watch(petRepositoryProvider);
  return repository.getPets(filter);
});

// ✅ Use AsyncValue for loading/error states
// ✅ Use .autoDispose to prevent memory leaks
// ✅ Use .family for parameterized providers
```

---

## 5. Next.js (Web + Admin) Conventions

### 5.1 App Router Structure

```
src/
├── app/
│   ├── layout.tsx          # Root layout
│   ├── page.tsx            # Homepage
│   ├── pets/
│   │   ├── page.tsx        # Pet listings (SSR)
│   │   └── [slug]/
│   │       └── page.tsx    # Pet detail (SSR)
│   ├── products/
│   ├── cart/
│   ├── checkout/
│   ├── account/
│   └── admin/              # Admin panel (separate layout)
├── components/
│   ├── ui/                 # Primitive components (shadcn)
│   ├── layout/             # Header, Footer, Sidebar
│   ├── pets/               # Pet-specific components
│   ├── products/           # Product-specific components
│   └── shared/             # Shared across features
├── lib/
│   ├── api/                # API client functions
│   ├── hooks/              # Custom React hooks
│   ├── utils/              # Utility functions
│   └── validations/        # Zod schemas
├── types/                  # TypeScript types/interfaces
└── styles/
    └── globals.css         # Tailwind base
```

### 5.2 Component Rules

```tsx
// ✅ Server Components by default (no "use client" unless needed)
// ✅ "use client" only for interactivity (onClick, useState, hooks)

// ✅ Name components same as file (PascalCase)
// File: components/pets/pet-card.tsx
export function PetCard({ pet }: { pet: Pet }) {
  return (
    <div className="rounded-lg border p-4">
      <Image src={pet.image} alt={pet.breed} />
      <h3>{pet.breed}</h3>
      <p>₹{pet.price.toLocaleString('en-IN')}</p>
    </div>
  );
}

// ✅ Use Zod for form validation
const createPetSchema = z.object({
  species: z.enum(['dog', 'cat', 'bird']),
  breed: z.string().min(2).max(100),
  price: z.number().min(0),
});
```

---

## 6. Database Conventions

```sql
-- ✅ Table names: plural snake_case
CREATE TABLE pet_listings (...)
CREATE TABLE order_items (...)

-- ✅ Primary key: always `id` (UUID)
id UUID PRIMARY KEY DEFAULT gen_random_uuid()

-- ✅ Foreign keys: <singular_table>_id
seller_id UUID REFERENCES users(id)

-- ✅ Timestamps on every table
created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

-- ✅ Soft delete where applicable
deleted_at TIMESTAMPTZ NULL

-- ✅ Index naming: idx_<table>_<columns>
CREATE INDEX idx_pet_listings_species_city ON pet_listings(species, city);

-- ❌ Never store passwords in plain text
-- ❌ Never store sensitive data unencrypted
```

---

## 7. API Response Format

### Success Response

```json
{
  "data": { ... },
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

### Error Response

```json
{
  "statusCode": 400,
  "error": "VALIDATION_ERROR",
  "message": "Price must be a positive number",
  "details": [
    { "field": "price", "message": "Must be greater than 0" }
  ],
  "timestamp": "2026-05-28T10:30:00Z",
  "path": "/api/v1/pets"
}
```

---

## 8. Error Handling

```typescript
// ✅ Use custom exception classes
export class PetNotFoundException extends NotFoundException {
  constructor(id: string) {
    super({ error: 'PET_NOT_FOUND', message: `Pet ${id} not found` });
  }
}

// ✅ Global exception filter handles all errors uniformly
// ✅ Never expose stack traces in production
// ✅ Log full error internally, return safe message to client
```

---

## 9. Security Rules

- **Never** commit secrets (`.env` files, API keys) to git
- **Never** log sensitive data (passwords, tokens, card numbers)
- **Always** validate and sanitize user input
- **Always** use parameterized queries (Prisma handles this)
- **Always** check authorization (user owns the resource)
- **Always** rate-limit public endpoints
- **Never** trust client-side validation alone — validate server-side too

---

## 10. Comments & Documentation

```typescript
// ✅ Use JSDoc for public methods that aren't self-explanatory
/**
 * Releases escrow payment to seller after buyer confirms pet receipt.
 * @throws ConflictException if escrow already released
 */
async releaseEscrow(orderId: string): Promise<void> { }

// ✅ Use TODO with ticket number for planned work
// TODO(PZ-123): Add support for bulk photo upload

// ❌ Don't write comments that restate the code
// Bad: // increment counter by 1
// count++;
```

---

## 11. Tooling

| Tool | Purpose | Config File |
|------|---------|-------------|
| ESLint | Linting (TS/JS) | `.eslintrc.js` |
| Prettier | Formatting | `.prettierrc` |
| Husky | Git hooks | `.husky/` |
| lint-staged | Pre-commit checks | `package.json` |
| commitlint | Commit message format | `commitlint.config.js` |
| dart_linter | Flutter linting | `analysis_options.yaml` |

### Commit Message Format (Conventional Commits)

```
<type>(<scope>): <short description>

Types: feat, fix, docs, style, refactor, test, chore, perf
Scope: auth, pets, products, orders, payments, chat, admin, infra

Examples:
feat(pets): add breed filter to search API
fix(orders): correct escrow release timing
docs(api): update payment webhook documentation
refactor(auth): extract OTP logic to separate service
```
