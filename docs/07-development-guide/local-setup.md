# PetZonic — Local Development Setup

> **Version**: 1.0.0  
> **Date**: May 28, 2026

---

## 1. Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Node.js | 22.x LTS | [nodejs.org](https://nodejs.org/) or `nvm install 22` |
| npm | 10.x | Included with Node.js 22 |
| Docker Desktop | Latest | [docker.com](https://www.docker.com/products/docker-desktop/) |
| Git | Latest | [git-scm.com](https://git-scm.com/) |
| VS Code | Latest | [code.visualstudio.com](https://code.visualstudio.com/) |

> Flutter, Android Studio and Xcode are **not** required. The mobile apps do not exist —
> `petzonic-customer-app` and `petzonic-seller-app` contain no application code.

---

## 2. Clone Repositories

```bash
# Create project directory
mkdir petzonic && cd petzonic

# Clone the repos that contain code
git clone git@github.com:petZonic/petzonic-api.git
git clone git@github.com:petZonic/petzonic-web.git
git clone git@github.com:petZonic/petzonic-admin.git
git clone git@github.com:petZonic/petzonic-infra.git

# Documentation (this repo)
git clone git@github.com:sudarsan-22/PetZonic.git

# NOTE: petzonic-customer-app and petzonic-seller-app are empty stubs containing no
# application code. You do not need them for local development.
```

> **All repos must be cloned as siblings inside one parent directory.** `petzonic-infra`
> builds the others through relative paths (`../../petzonic-api`), so its Docker Compose files
> assume this exact layout.
>
> **Package manager: npm.** Every repo commits a `package-lock.json`. Do not use pnpm or
> yarn — earlier CI workflows wrongly assumed pnpm and had to be corrected.

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
| Redis 7 | 6379 | Rate limiting, distributed lock & AI session store |
| Ollama | 11434 | Local LLM inference engine (`qwen2.5:3b`) |
| Backend API | 4000 | Express 5 REST API + WebSockets |
| Frontend Web | 3001 | Customer Next.js storefront |
| Admin Panel | 3002 | Administrative management panel |

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

# AI Shopping Discovery Chatbot
AI_DISCOVERY_ENABLED=true
AI_DISCOVERY_ROLLOUT_PERCENTAGE=15
AI_DISCOVERY_PROVIDER=ollama
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=qwen2.5:3b
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

# Run full Vitest integration suite (46 test files as of 2026-09-20)
npm run test:run

# Run tests with code coverage report
npm run test:coverage
```

---

## 4. Customer App Setup (Flutter) — 📋 NOT APPLICABLE

> **Skip this section.** `petzonic-customer-app` is an empty stub with no Flutter project,
> no `pubspec.yaml` and no Dart code. Every command below will fail. Retained for when mobile
> work actually begins.

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

## 5. Seller App Setup (Flutter) — 📋 NOT APPLICABLE

> **Skip this section.** `petzonic-seller-app` is an empty stub. The seller experience that
> exists today is in `petzonic-web` at `/seller/*` and needs no separate setup.

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
npm install

# Configure environment
cp .env.local.example .env.local
```

> **Read `petzonic-web/AGENTS.md` before editing this app.** It warns that its Next.js 16
> setup diverges from widely-known Next.js patterns and points to the bundled docs under
> `node_modules/next/dist/docs/`.

### Admin Panel Setup

```bash
cd petzonic-admin
npm install
cp .env.local.example .env.local
npm run dev   # serves on port 3002
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

# Website at http://localhost:3001 (customer + /seller/* + /provider/* portals)
# Admin panel is a SEPARATE app: http://localhost:3002 (see Admin Panel Setup above)
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
| `npm run test:run` | Run full Vitest integration suite (46 test files) |
| `npm run test:coverage` | Run tests with V8 coverage report |
| `npm run build` | Compile TypeScript into `dist/` |
| `npm run db:push` | Sync Prisma schema with database |
| `npm run db:seed` | Seed test users, listings, products, & services |

### Frontend (`petzonic-web`)

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start Next.js dev server on port 3001 |
| `npm run test:run` | Run Vitest component tests (99 test files) |
| `npm run lint` | Run ESLint code quality check |
| `npm run build` | Next.js production build (85 page routes as of 2026-09-20) |

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
| Port 4000 already in use | Kill process: `lsof -ti:4000 \| xargs kill` or change PORT in .env |
| Prisma migration fails | `npx prisma migrate reset` (drops DB, re-runs all) |
| Flutter build fails | `flutter clean && flutter pub get` |
| Docker OOM | Increase Docker Desktop memory (≥4GB recommended) |
| iOS pod install fails | `cd ios && pod deintegrate && pod install` |
| Android Gradle sync fails | `cd android && ./gradlew clean` |
| pg_trgm extension missing | Connect to Postgres and run `CREATE EXTENSION IF NOT EXISTS pg_trgm;` |
| Redis connection refused | `docker compose restart redis` |
