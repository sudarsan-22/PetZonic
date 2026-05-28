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
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@prisma/client';

// 3. Internal modules (absolute paths)
import { UserService } from '@/modules/user/user.service';
import { PaginationDto } from '@/common/dto/pagination.dto';

// 4. Relative imports (same module)
import { CreatePetDto } from './dto/create-pet.dto';
```

---

## 3. NestJS (Backend) Conventions

### 3.1 Module Structure

```
src/
├── main.ts
├── app.module.ts
├── common/                     # Shared utilities
│   ├── decorators/
│   ├── dto/
│   ├── exceptions/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   ├── pipes/
│   └── utils/
├── config/                     # Configuration
│   ├── app.config.ts
│   ├── database.config.ts
│   └── redis.config.ts
├── modules/                    # Feature modules
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.guard.ts
│   │   ├── dto/
│   │   │   ├── login.dto.ts
│   │   │   └── register.dto.ts
│   │   ├── entities/
│   │   └── tests/
│   │       ├── auth.service.spec.ts
│   │       └── auth.controller.spec.ts
│   ├── pets/
│   ├── products/
│   ├── orders/
│   └── ...
└── prisma/
    ├── schema.prisma
    ├── migrations/
    └── seed.ts
```

### 3.2 Controller Rules

```typescript
@Controller('pets')
@ApiTags('Pets')
export class PetsController {
  constructor(private readonly petsService: PetsService) {}

  // ✅ Use HTTP decorators explicitly
  @Get()
  @ApiOperation({ summary: 'List pet listings' })
  async findAll(@Query() query: ListPetsDto): Promise<PaginatedResponse<Pet>> {
    return this.petsService.findAll(query);
  }

  // ✅ Use ParseUUIDPipe for ID params
  @Get(':id')
  async findOne(@Param('id', ParseUUIDPipe) id: string): Promise<Pet> {
    return this.petsService.findOne(id);
  }

  // ✅ Use DTOs for request validation
  @Post()
  @UseGuards(JwtAuthGuard)
  async create(
    @Body() dto: CreatePetDto,
    @CurrentUser() user: User,
  ): Promise<Pet> {
    return this.petsService.create(dto, user.id);
  }
}
```

### 3.3 Service Rules

```typescript
@Injectable()
export class PetsService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly searchService: SearchService,
  ) {}

  // ✅ Business logic lives in services, not controllers
  // ✅ Throw specific HTTP exceptions
  async findOne(id: string): Promise<Pet> {
    const pet = await this.prisma.pet.findUnique({ where: { id } });
    if (!pet) {
      throw new NotFoundException(`Pet with ID ${id} not found`);
    }
    return pet;
  }

  // ✅ Use transactions for multi-table operations
  async createOrder(dto: CreateOrderDto, userId: string): Promise<Order> {
    return this.prisma.$transaction(async (tx) => {
      const order = await tx.order.create({ ... });
      await tx.orderItem.createMany({ ... });
      await tx.inventory.update({ ... });
      return order;
    });
  }
}
```

### 3.4 DTO Validation

```typescript
// ✅ Use class-validator decorators
export class CreatePetDto {
  @IsEnum(Species)
  species: Species;

  @IsString()
  @MinLength(2)
  @MaxLength(100)
  breed: string;

  @IsNumber()
  @Min(0)
  @Max(10000000) // ₹1 Crore max
  price: number;

  @IsOptional()
  @IsString()
  @MaxLength(2000)
  description?: string;

  @IsArray()
  @ArrayMinSize(3)
  @ArrayMaxSize(10)
  @IsUrl({}, { each: true })
  images: string[];
}
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
