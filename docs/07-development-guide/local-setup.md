# PetZonic — Local Development Setup

> **Version**: 1.0.0  
> **Date**: May 28, 2026

---

## 1. Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Node.js | 22.x LTS | [nodejs.org](https://nodejs.org/) or `nvm install 22` |
| npm | 10.x | Included with Node.js 22 |
| Flutter | 3.x | [flutter.dev](https://flutter.dev/docs/get-started/install) |
| Docker Desktop | Latest | [docker.com](https://www.docker.com/products/docker-desktop/) |
| Git | Latest | [git-scm.com](https://git-scm.com/) |
| VS Code | Latest | [code.visualstudio.com](https://code.visualstudio.com/) |
| Android Studio | Latest | For Android emulator + SDK |
| Xcode | 15+ (macOS) | For iOS simulator |

---

## 2. Clone Repositories

```bash
# Create project directory
mkdir petzonic && cd petzonic

# Clone all repos
git clone git@github.com:petzonic/petzonic-api.git
git clone git@github.com:petzonic/petzonic-customer-app.git
git clone git@github.com:petzonic/petzonic-seller-app.git
git clone git@github.com:petzonic/petzonic-web.git
git clone git@github.com:petzonic/petzonic-infra.git
```

---

## 3. Backend API Setup

### 3.1 Start Infrastructure (Docker)

```bash
cd petzonic-infra/"Deployment container"

# Start PostgreSQL and Redis
docker compose up -d postgres redis

# Verify services are running
docker compose ps
```

**Docker Compose services**:

| Service | Port | Purpose |
|---------|:----:|---------|
| PostgreSQL 16 | 5432 | Primary database with `pg_trgm` fuzzy search |
| Redis 7 | 6379 | Rate limiting, distributed lock & cache store |

### 3.2 Install Dependencies & Configure

```bash
cd ../../petzonic-api

# Install Node packages
npm install

# Copy environment file
cp .env.example .env
```

### 3.3 Environment Variables (`.env`)

```env
# Database
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/petzonic?schema=public"

# Redis
REDIS_URL="redis://localhost:6379"

# JWT
JWT_SECRET=dev-secret-change-in-production-long-random-string
JWT_REFRESH_SECRET=dev-refresh-secret-change-in-production-long-random-string

# App & CORS
PORT=4000
NODE_ENV=development
CLIENT_URL=http://localhost:3001

# Cloud Storage (Optional - falls back to local /uploads if omitted)
# AWS_ACCESS_KEY_ID=
# AWS_SECRET_ACCESS_KEY=
# AWS_REGION=ap-south-1
# AWS_S3_BUCKET=petzonic-media

# Razorpay (Optional in development)
# RAZORPAY_KEY_ID=rzp_test_xxxxx
# RAZORPAY_KEY_SECRET=xxxxx

# Gemini AI (Optional - returns honest 503 fallback if omitted)
# GEMINI_API_KEY=
```

### 3.4 Database Setup

```bash
# Generate Prisma client
npm run db:generate

# Push schema to database
npm run db:push

# Seed database with sample data
npm run db:seed
```

### 3.5 Run API Server

```bash
# Development mode (with live watch)
npm run dev

# API running at http://localhost:4000
# Health check at http://localhost:4000/api/health
# Interactive Swagger docs at http://localhost:4000/api/docs
```

### 3.6 Run Tests

```bash
# Static type safety check
npm run typecheck

# Run full Vitest integration suite (28 test files, 445 tests)
npm run test:run

# Run tests with code coverage report
npm run test:coverage
```

---

## 4. Customer App Setup (Flutter)

```bash
cd petzonic-customer-app

# Get Flutter dependencies
flutter pub get

# Generate code (Riverpod, json_serializable, etc.)
dart run build_runner build --delete-conflicting-outputs

# Configure API base URL
cp .env.example .env
# Edit: API_BASE_URL=http://localhost:3000/api/v1
# For Android emulator: API_BASE_URL=http://10.0.2.2:3000/api/v1

# Run on device/emulator
flutter run

# Run on specific device
flutter devices              # List available devices
flutter run -d <device_id>

# Run with hot reload
# (already enabled by default with `flutter run`)
```

### iOS Specific (macOS only)

```bash
cd ios
pod install
cd ..
flutter run -d iPhone   # Or specific simulator
```

### Android Specific

```bash
# Ensure Android SDK is configured
flutter doctor

# Run on emulator
flutter emulators --launch <emulator_name>
flutter run
```

---

## 5. Seller App Setup (Flutter)

```bash
cd petzonic-seller-app

# Same setup as customer app
flutter pub get
dart run build_runner build --delete-conflicting-outputs
cp .env.example .env
flutter run
```

---

## 6. Website Setup (Next.js)

```bash
cd petzonic-web

# Install dependencies
pnpm install

# Configure environment
cp .env.local.example .env.local
```

### Environment (.env.local)

```env
NEXT_PUBLIC_API_URL=http://localhost:4000/api/v1
NEXT_PUBLIC_WS_URL=ws://localhost:4000
NEXT_PUBLIC_RAZORPAY_KEY=rzp_test_xxxxx
NEXT_PUBLIC_SITE_URL=http://localhost:3001
```

```bash
# Run development server
npm run dev

# Website at http://localhost:3001
# Admin panel at http://localhost:3001/admin
```

---

## 7. VS Code Extensions (Recommended)

### Backend & Frontend (TypeScript / React)
- ESLint
- Prettier
- Prisma
- Tailwind CSS IntelliSense
- GitLens
- Docker

### Flutter
- Flutter
- Dart
- Flutter Riverpod Snippets

---

## 8. VS Code Settings (Workspace)

Create `.vscode/settings.json` in each repo:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit",
    "source.organizeImports": "explicit"
  },
  "typescript.preferences.importModuleSpecifier": "non-relative"
}
```

---

## 9. Useful Commands Quick Reference

### Backend (`petzonic-api`)

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start API in dev mode with live watch |
| `npm run typecheck` | Static TypeScript type safety check |
| `npm run test:run` | Run full Vitest integration suite (445 tests) |
| `npm run test:coverage` | Run tests with V8 coverage report |
| `npm run build` | Compile TypeScript into `dist/` |
| `npm run db:push` | Sync Prisma schema with database |
| `npm run db:seed` | Seed test users, listings, products, & services |

### Frontend (`petzonic-web`)

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start Next.js dev server on port 3001 |
| `npm run test:run` | Run Vitest component tests (531 tests) |
| `npm run lint` | Run ESLint code quality check |
| `npm run build` | Next.js production build (79 routes) |

---

## 10. Seed Data

After running `npm run db:seed`, these test accounts are seeded in the database:

| Role | Email | Password |
|------|-------|----------|
| Admin | admin@example.com | admin123 |
| Buyer | buyer@example.com | buyer123 |
| Seller | seller@example.com | seller123 |

**Sample data created**:
- 4 Species (Dogs, Cats, Birds, Fish) & 14 Breeds
- 8 Product Categories & Seed Products
- Pet Listings with images and pricing
- Service Providers & Bookable Services
- Insurance Partners & Coverage Plans

---

## 11. Troubleshooting

| Issue | Solution |
|-------|---------|
| Port 5432 already in use | Stop local Postgres: `sudo service postgresql stop` |
| Port 3000 already in use | Kill process: `lsof -ti:3000 \| xargs kill` or change PORT in .env |
| Prisma migration fails | `pnpm prisma migrate reset` (drops DB, re-runs all) |
| Flutter build fails | `flutter clean && flutter pub get` |
| Docker OOM | Increase Docker Desktop memory (≥4GB recommended) |
| iOS pod install fails | `cd ios && pod deintegrate && pod install` |
| Android Gradle sync fails | `cd android && ./gradlew clean` |
| Meilisearch not indexing | Check master key matches in .env |
| Redis connection refused | `docker compose restart redis` |
