# PetZonic — Local Development Setup

> **Version**: 1.0.0  
> **Date**: May 28, 2026

---

## 1. Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Node.js | 22.x LTS | [nodejs.org](https://nodejs.org/) or `nvm install 22` |
| pnpm | 9.x | `npm install -g pnpm` |
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
cd petzonic-api

# Start PostgreSQL, Redis, Meilisearch via Docker Compose
docker compose up -d

# Verify services are running
docker compose ps
```

**Docker Compose services**:

| Service | Port | Purpose |
|---------|:----:|---------|
| PostgreSQL 16 | 5432 | Primary database |
| Redis 7 | 6379 | Cache, sessions, queues |
| Meilisearch | 7700 | Full-text search |
| MailHog | 8025 | Email testing (catches all outbound email) |

### 3.2 Install Dependencies & Configure

```bash
# Install Node packages
pnpm install

# Copy environment file
cp .env.example .env

# Edit .env with your local values (defaults work for Docker services)
```

### 3.3 Environment Variables (.env)

```env
# Database
DATABASE_URL="postgresql://petzonic:petzonic@localhost:5432/petzonic_dev"

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# JWT
JWT_SECRET=local-dev-secret-change-in-production
JWT_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

# Meilisearch
MEILISEARCH_HOST=http://localhost:7700
MEILISEARCH_KEY=local-master-key

# Razorpay (sandbox)
RAZORPAY_KEY_ID=rzp_test_xxxxx
RAZORPAY_KEY_SECRET=xxxxx

# AWS S3 (use LocalStack or MinIO for local)
AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=localdev
AWS_SECRET_ACCESS_KEY=localdev
S3_BUCKET=petzonic-dev
S3_ENDPOINT=http://localhost:9000  # MinIO

# Firebase (for push notifications - skip in local dev)
# FIREBASE_PROJECT_ID=
# FIREBASE_PRIVATE_KEY=

# App
PORT=3000
NODE_ENV=development
API_PREFIX=api/v1
```

### 3.4 Database Setup

```bash
# Generate Prisma client
pnpm prisma generate

# Run migrations
pnpm prisma migrate dev

# Seed database with sample data
pnpm prisma db seed

# (Optional) Open Prisma Studio to browse data
pnpm prisma studio
# Opens at http://localhost:5555
```

### 3.5 Run API Server

```bash
# Development mode (hot reload)
pnpm run start:dev

# API running at http://localhost:3000
# Swagger docs at http://localhost:3000/api/docs
```

### 3.6 Run Tests

```bash
# Unit tests
pnpm test

# Unit tests (watch mode)
pnpm test:watch

# E2E tests
pnpm test:e2e

# Test coverage
pnpm test:cov
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
NEXT_PUBLIC_API_URL=http://localhost:3000/api/v1
NEXT_PUBLIC_WS_URL=ws://localhost:3000
NEXT_PUBLIC_RAZORPAY_KEY=rzp_test_xxxxx
NEXT_PUBLIC_SITE_URL=http://localhost:3001
```

```bash
# Run development server
pnpm dev

# Website at http://localhost:3001
# Admin panel at http://localhost:3001/admin
```

---

## 7. VS Code Extensions (Recommended)

### Backend (NestJS/TypeScript)
- ESLint
- Prettier
- Prisma
- REST Client (or Thunder Client)
- GitLens
- Error Lens
- Docker

### Flutter
- Flutter
- Dart
- Flutter Riverpod Snippets
- Awesome Flutter Snippets

### Frontend (Next.js)
- Tailwind CSS IntelliSense
- ES7+ React Snippets
- Auto Rename Tag

### General
- GitHub Copilot
- Material Icon Theme
- Todo Tree

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
  "typescript.preferences.importModuleSpecifier": "non-relative",
  "[dart]": {
    "editor.defaultFormatter": "Dart-Code.dart-code",
    "editor.formatOnSave": true,
    "editor.selectionHighlight": false,
    "editor.rulers": [80]
  }
}
```

---

## 9. Useful Commands Quick Reference

### Backend

| Command | Purpose |
|---------|---------|
| `pnpm start:dev` | Start API (dev mode) |
| `pnpm test` | Run unit tests |
| `pnpm test:e2e` | Run E2E tests |
| `pnpm prisma studio` | Browse database |
| `pnpm prisma migrate dev` | Create/apply migration |
| `pnpm prisma db seed` | Seed sample data |
| `pnpm lint` | Run ESLint |
| `pnpm format` | Run Prettier |
| `docker compose up -d` | Start infra services |
| `docker compose down` | Stop infra services |
| `docker compose logs -f postgres` | View Postgres logs |

### Flutter

| Command | Purpose |
|---------|---------|
| `flutter run` | Run app on device |
| `flutter test` | Run tests |
| `flutter analyze` | Static analysis |
| `dart run build_runner build` | Code generation |
| `flutter clean` | Clean build cache |
| `flutter pub upgrade` | Upgrade packages |
| `flutter build apk` | Build Android APK |
| `flutter build ios` | Build iOS |

### Next.js

| Command | Purpose |
|---------|---------|
| `pnpm dev` | Start dev server |
| `pnpm build` | Production build |
| `pnpm start` | Start production server |
| `pnpm lint` | Run ESLint |
| `pnpm test` | Run tests |

---

## 10. Seed Data

After running `pnpm prisma db seed`, these test accounts are available:

| Role | Email/Phone | Password/OTP |
|------|------------|--------------|
| Admin | admin@petzonic.com | admin123 |
| Buyer | +91-9876543210 | 123456 (fixed in dev) |
| Seller | +91-9876543211 | 123456 |
| Breeder | +91-9876543212 | 123456 |
| Vet | +91-9876543213 | 123456 |

**Sample data created**:
- 20 pet listings (various species, cities)
- 50 products (food, accessories, health)
- 5 service providers (vets, groomers)
- 3 categories with subcategories
- Sample orders, reviews, chats

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
