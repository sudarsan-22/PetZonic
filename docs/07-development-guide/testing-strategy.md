# PetZonic — Testing Strategy

> **Version**: 1.0.0  
> **Date**: May 28, 2026

---

## 1. Testing Philosophy

- **Test behavior, not implementation** — Tests should survive refactoring.
- **Test at the right level** — Unit tests for logic, integration tests for flows, E2E for critical paths.
- **Fast feedback** — Unit tests run in < 30 seconds. CI completes in < 5 minutes.
- **No flaky tests** — If a test fails intermittently, fix or remove it immediately.

---

## 2. Testing Pyramid

```
         /‾‾‾‾‾‾\
        / E2E (5%) \         ← Few, slow, high confidence
       /────────────\
      / Integration  \       ← Moderate count, test module boundaries
     / (20%)          \
    /──────────────────\
   / Unit Tests (75%)   \   ← Many, fast, isolated
  /________________________\
```

| Level | What to Test | Tools | Speed |
|-------|-------------|-------|:-----:|
| Unit | Business logic, utils, pure functions | Jest, flutter_test | Fast |
| Integration | API endpoints, DB queries, services together | Supertest, Prisma test DB | Medium |
| E2E | Critical user journeys across the full stack | Detox (mobile), Playwright (web) | Slow |

---

## 3. Coverage Targets

| Area | Target | Enforced |
|------|:------:|:--------:|
| Backend services | ≥ 80% | CI gate |
| Backend controllers | ≥ 70% | CI gate |
| Flutter business logic | ≥ 75% | CI gate |
| Flutter widgets | ≥ 50% | Advisory |
| Next.js utilities | ≥ 80% | CI gate |
| Next.js components | ≥ 50% | Advisory |
| E2E critical paths | 100% of P0 flows | Manual check |

---

## 4. Backend Testing (NestJS + Jest)

### 4.1 Unit Tests

Test services and utilities in isolation with mocked dependencies.

```typescript
// pets.service.spec.ts
describe('PetsService', () => {
  let service: PetsService;
  let prisma: DeepMockProxy<PrismaClient>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        PetsService,
        { provide: PrismaService, useValue: mockDeep<PrismaClient>() },
        { provide: SearchService, useValue: { indexPet: jest.fn() } },
      ],
    }).compile();

    service = module.get(PetsService);
    prisma = module.get(PrismaService);
  });

  describe('findOne', () => {
    it('should return pet when found', async () => {
      const mockPet = { id: '1', breed: 'Golden Retriever', price: 25000 };
      prisma.pet.findUnique.mockResolvedValue(mockPet);

      const result = await service.findOne('1');
      expect(result).toEqual(mockPet);
    });

    it('should throw NotFoundException when pet not found', async () => {
      prisma.pet.findUnique.mockResolvedValue(null);

      await expect(service.findOne('999')).rejects.toThrow(NotFoundException);
    });
  });

  describe('create', () => {
    it('should create pet and index in search', async () => {
      // ...
    });

    it('should validate minimum 3 images', async () => {
      // ...
    });
  });
});
```

### 4.2 Integration Tests (E2E with real DB)

Test full request → response cycle with a test database.

```typescript
// pets.e2e-spec.ts
describe('PetsController (e2e)', () => {
  let app: INestApplication;
  let prisma: PrismaService;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = module.createNestApplication();
    app.useGlobalPipes(new ValidationPipe());
    await app.init();

    prisma = app.get(PrismaService);
  });

  beforeEach(async () => {
    // Clean test DB between tests
    await prisma.pet.deleteMany();
    await prisma.user.deleteMany();
  });

  afterAll(async () => {
    await app.close();
  });

  describe('POST /api/v1/pets', () => {
    it('should create pet listing with valid data', async () => {
      const token = await getAuthToken(app, 'seller');

      const response = await request(app.getHttpServer())
        .post('/api/v1/pets')
        .set('Authorization', `Bearer ${token}`)
        .send({
          species: 'dog',
          breed: 'Golden Retriever',
          price: 25000,
          gender: 'male',
          age: { value: 3, unit: 'months' },
          images: ['img1.jpg', 'img2.jpg', 'img3.jpg'],
          city: 'Bangalore',
        })
        .expect(201);

      expect(response.body.id).toBeDefined();
      expect(response.body.status).toBe('PENDING_REVIEW');
    });

    it('should reject with less than 3 images', async () => {
      const token = await getAuthToken(app, 'seller');

      await request(app.getHttpServer())
        .post('/api/v1/pets')
        .set('Authorization', `Bearer ${token}`)
        .send({
          species: 'dog',
          breed: 'Golden Retriever',
          price: 25000,
          images: ['img1.jpg'],
        })
        .expect(400);
    });

    it('should require authentication', async () => {
      await request(app.getHttpServer())
        .post('/api/v1/pets')
        .send({ species: 'dog' })
        .expect(401);
    });
  });
});
```

### 4.3 Test Database

```bash
# Docker Compose includes a test DB
# DATABASE_URL for tests: postgresql://petzonic:petzonic@localhost:5433/petzonic_test

# Run migrations on test DB before tests
DATABASE_URL="postgresql://..." pnpm prisma migrate deploy
```

---

## 5. Mobile Testing (Flutter)

### 5.1 Unit Tests

```dart
// pet_repository_test.dart
void main() {
  late PetRepository repository;
  late MockApiClient mockApiClient;

  setUp(() {
    mockApiClient = MockApiClient();
    repository = PetRepository(apiClient: mockApiClient);
  });

  group('getPets', () {
    test('returns list of pets on success', () async {
      when(mockApiClient.get('/pets')).thenAnswer(
        (_) async => Response(data: mockPetListJson, statusCode: 200),
      );

      final pets = await repository.getPets(PetFilter());
      
      expect(pets, isA<List<Pet>>());
      expect(pets.length, 2);
      expect(pets.first.breed, 'Golden Retriever');
    });

    test('throws exception on network error', () async {
      when(mockApiClient.get('/pets')).thenThrow(
        DioException(type: DioExceptionType.connectionTimeout),
      );

      expect(
        () => repository.getPets(PetFilter()),
        throwsA(isA<NetworkException>()),
      );
    });
  });
}
```

### 5.2 Widget Tests

```dart
// pet_card_test.dart
void main() {
  testWidgets('PetCard displays breed and price', (tester) async {
    final pet = Pet(
      id: '1',
      breed: 'Labrador',
      price: 15000,
      images: ['test.jpg'],
    );

    await tester.pumpWidget(
      MaterialApp(home: Scaffold(body: PetCard(pet: pet))),
    );

    expect(find.text('Labrador'), findsOneWidget);
    expect(find.text('₹15,000'), findsOneWidget);
  });

  testWidgets('PetCard navigates to detail on tap', (tester) async {
    // ...
  });
}
```

### 5.3 Integration Tests (Detox / patrol)

```dart
// test/integration/purchase_flow_test.dart
void main() {
  patrolTest('Complete product purchase flow', ($) async {
    // Login
    await $.pumpWidgetAndSettle(const PetZonicApp());
    await $(#phoneInput).enterText('9876543210');
    await $(#sendOtpButton).tap();
    await $(#otpInput).enterText('123456');
    await $(#verifyButton).tap();

    // Browse products
    await $(#productsTab).tap();
    await $(#productCard).first.tap();

    // Add to cart
    await $(#addToCartButton).tap();
    expect($(#cartBadge).text, '1');

    // Checkout
    await $(#cartIcon).tap();
    await $(#checkoutButton).tap();
    await $(#selectAddress).first.tap();
    await $(#placeOrderButton).tap();

    // Verify success
    expect($(#orderSuccessScreen), findsOneWidget);
  });
}
```

---

## 6. Web Testing (Next.js)

### 6.1 Component Tests (React Testing Library)

```typescript
// PetCard.test.tsx
describe('PetCard', () => {
  it('renders pet breed and price', () => {
    render(<PetCard pet={mockPet} />);
    
    expect(screen.getByText('Golden Retriever')).toBeInTheDocument();
    expect(screen.getByText('₹25,000')).toBeInTheDocument();
  });

  it('shows verified badge for verified sellers', () => {
    render(<PetCard pet={{ ...mockPet, seller: { isVerified: true } }} />);
    
    expect(screen.getByLabelText('Verified seller')).toBeInTheDocument();
  });
});
```

### 6.2 E2E Tests (Playwright)

```typescript
// tests/checkout.spec.ts
test('complete checkout flow', async ({ page }) => {
  await page.goto('/products');
  await page.click('[data-testid="product-card"]:first-child');
  await page.click('[data-testid="add-to-cart"]');
  await page.goto('/cart');
  await page.click('[data-testid="checkout-button"]');
  
  // Select address
  await page.click('[data-testid="address-card"]:first-child');
  await page.click('[data-testid="place-order"]');
  
  // Verify order confirmation
  await expect(page.locator('[data-testid="order-success"]')).toBeVisible();
});
```

---

## 7. API Testing (Postman/Newman)

### Collection Structure

```
PetZonic API/
├── Auth/
│   ├── Register with OTP
│   ├── Login with OTP
│   ├── Refresh Token
│   └── Logout
├── Pets/
│   ├── List Pets
│   ├── Get Pet by ID
│   ├── Create Pet (Seller)
│   ├── Update Pet (Seller)
│   └── Delete Pet (Seller)
├── Products/
├── Orders/
├── Payments/
├── Chat/
└── Admin/
```

### CI Integration

```bash
# Run Postman collection in CI
npx newman run postman/collection.json \
  --environment postman/staging.env.json \
  --reporters cli,junit \
  --reporter-junit-export results.xml
```

---

## 8. Performance Testing

### Tool: k6

```javascript
// load-test/pet-search.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 50 },    // Ramp up
    { duration: '3m', target: 200 },   // Sustained load
    { duration: '1m', target: 1000 },  // Spike
    { duration: '1m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95th percentile < 500ms
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
  },
};

export default function () {
  const res = http.get('https://api.petzonic.com/api/v1/pets?species=dog&city=bangalore');
  
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  sleep(1);
}
```

### Performance Targets

| Endpoint | p50 | p95 | p99 |
|----------|:---:|:---:|:---:|
| GET /pets (listing) | < 100ms | < 300ms | < 500ms |
| GET /pets/:id | < 50ms | < 150ms | < 300ms |
| POST /orders | < 200ms | < 500ms | < 1000ms |
| GET /products/search | < 50ms | < 200ms | < 400ms |
| WebSocket message | < 100ms | < 200ms | < 500ms |

---

## 9. Security Testing

| Test Type | Tool | When |
|-----------|------|------|
| Dependency vulnerabilities | `npm audit`, Snyk | Every CI run |
| OWASP Top 10 scan | OWASP ZAP | Before each release |
| SQL injection | Automated (Prisma prevents) | Integration tests |
| XSS testing | Playwright + manual | Before each release |
| Auth bypass | Manual penetration test | Before launch |
| Rate limiting | k6 | Before launch |

---

## 10. Test Data Management

### Factories (Backend)

```typescript
// test/factories/pet.factory.ts
export function createPetFactory(overrides?: Partial<Pet>): Pet {
  return {
    id: faker.string.uuid(),
    species: 'dog',
    breed: faker.helpers.arrayElement(['Labrador', 'Golden Retriever', 'Pug']),
    price: faker.number.int({ min: 5000, max: 100000 }),
    gender: faker.helpers.arrayElement(['male', 'female']),
    city: faker.helpers.arrayElement(['Bangalore', 'Mumbai', 'Delhi']),
    status: 'ACTIVE',
    sellerId: faker.string.uuid(),
    images: [faker.image.url(), faker.image.url(), faker.image.url()],
    createdAt: faker.date.recent(),
    ...overrides,
  };
}
```

### Test Isolation Rules

- Each test file gets a clean database state
- Use transactions that rollback after each test (fast)
- Never depend on test execution order
- Never share mutable state between tests

---

## 11. CI Test Pipeline

```yaml
# .github/workflows/test.yml
name: Test
on: [pull_request]

jobs:
  backend-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: petzonic_test
          POSTGRES_PASSWORD: test
        ports: ['5432:5432']
      redis:
        image: redis:7
        ports: ['6379:6379']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - run: pnpm install
      - run: pnpm prisma migrate deploy
      - run: pnpm test --coverage
      - run: pnpm test:e2e

  flutter-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with: { flutter-version: '3.x' }
      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test --coverage

  web-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - run: pnpm install
      - run: pnpm lint
      - run: pnpm test
      - run: pnpm build  # Ensure it builds
```
