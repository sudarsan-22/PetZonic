# PetZonic — Database Schema

> **Version**: 1.2.0 (Synchronized with Production Prisma Schema)  
> **Date**: September 2026  
> **Database Engine**: PostgreSQL 16 (Prisma ORM)  
> **Total Models**: 58 Production Models

---

## Complete Prisma Schema

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated/prisma"
}

datasource db {
  provider = "postgresql"
}

// ==========================================
// ENUMS
// ==========================================

enum UserStatus {
  ACTIVE
  SUSPENDED
  BANNED
  DEACTIVATED
}

enum Role {
  BUYER
  SELLER
  BREEDER
  ADMIN
}

enum Gender {
  MALE
  FEMALE
}

enum PetListingStatus {
  DRAFT
  PENDING_REVIEW
  ACTIVE
  PAUSED
  SOLD
  EXPIRED
  REJECTED
}

enum PriceType {
  FIXED
  NEGOTIABLE
}

enum OrderStatus {
  PENDING_PAYMENT
  CONFIRMED
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
}

enum PaymentStatus {
  CREATED
  CAPTURED
  FAILED
  REFUNDED
}

enum PaymentMethod {
  UPI
  CREDIT_CARD
  DEBIT_CARD
  NET_BANKING
  COD
}

enum PaymentEventStatus {
  CREATED
  CAPTURED
  FAILED
  REFUNDED
  PARTIALLY_REFUNDED
}

enum EscrowStatus {
  HELD
  RELEASED
  DISPUTED
  REFUNDED
}

enum ServiceType {
  VET_CONSULTATION
  GROOMING
  PET_SITTING
  PET_WALKING
  PET_BOARDING
  PET_TRAINING
}

enum BookingStatus {
  PENDING
  CONFIRMED
  COMPLETED
  CANCELLED
}

enum ServiceProviderStatus {
  PENDING
  APPROVED
  REJECTED
}

enum MessageType {
  TEXT
  IMAGE
}

// ==========================================
// USER DOMAIN
// ==========================================

model User {
  id               String     @id @default(uuid()) @db.Uuid
  email            String?    @unique @db.VarChar(255)
  phone            String?    @unique @db.VarChar(15)
  passwordHash     String?    @map("password_hash") @db.VarChar(255)
  firstName        String     @map("first_name") @db.VarChar(100)
  lastName         String?    @map("last_name") @db.VarChar(100)
  avatarUrl        String?    @map("avatar_url") @db.VarChar(500)
  city             String?    @db.VarChar(100)
  state            String?    @db.VarChar(100)
  status           UserStatus @default(ACTIVE)
  emailVerified    Boolean    @default(false) @map("email_verified")
  // Admin moderation (Phase 4b) -- suspension is time-bounded,
  // ban is treated as permanent/indefinite (no `bannedUntil`).
  suspendedUntil   DateTime?  @map("suspended_until")
  suspensionReason String?    @map("suspension_reason") @db.VarChar(500)
  banReason        String?    @map("ban_reason") @db.VarChar(500)
  createdAt        DateTime   @default(now()) @map("created_at")
  updatedAt        DateTime   @updatedAt @map("updated_at")

  roles                    UserRole[]
  petListings              PetListing[]
  orders                   Order[]
  addresses                Address[]
  reviews                  Review[]
  bookings                 Booking[]
  refreshTokens            RefreshToken[]
  passwordResetTokens      PasswordResetToken[]
  cart                     Cart?
  payments                 Payment[]
  bankAccount              SellerBankAccount?
  conversationsAsBuyer     Conversation[]          @relation("ConversationsAsBuyer")
  conversationsAsSeller    Conversation[]          @relation("ConversationsAsSeller")
  sentMessages             Message[]
  notifications            Notification[]
  notificationPreference   NotificationPreference?
  deviceTokens             DeviceToken[]
  ownedServiceProviders    ServiceProvider[]
  communityPosts           Post[]
  postVotes                PostVote[]
  postFollows              PostFollow[]
  communityReplies         Reply[]
  replyVotes               ReplyVote[]
  lostFoundPosts           LostFoundPost[]
  authoredEducationContent EducationContent[]
  contentLikes             ContentLike[]
  authoredCourses          Course[]
  courseEnrollments        CourseEnrollment[]
  courseProgress           CourseProgress[]
  consultationsAsPatient   VetConsultation[]       @relation("ConsultationsAsPatient")
  consultationsAsVet       VetConsultation[]       @relation("ConsultationsAsVet")
  vetQaAsked               VetQA[]                 @relation("VetQaAsker")
  vetQaAnswered            VetQA[]                 @relation("VetQaAnswerer")
  insurancePolicies        InsurancePolicy[]
  kycSubmissions           KycSubmission[]         @relation("KycSubmitter")
  kycReviews               KycSubmission[]         @relation("KycReviewer")
  disputesRaised           Dispute[]               @relation("DisputesRaised")
  disputesResolved         Dispute[]               @relation("DisputesResolved")
  payoutsReceived          Payout[]                @relation("SellerPayouts")
  settingsUpdated          PlatformSettings[]      @relation("SettingsUpdater")
  auditLogEntries          AuditLog[]              @relation("AuditActor")
  moderatedListings        PetListing[]            @relation("ListingModerator")

  @@map("users")
}

model UserRole {
  id     String @id @default(uuid()) @db.Uuid
  userId String @map("user_id") @db.Uuid
  role   Role

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([userId, role])
  @@map("user_roles")
}

// ==========================================
// AUTH SUPPORT (OTP, refresh tokens, password reset)
// ==========================================

model OtpCode {
  id        String   @id @default(uuid()) @db.Uuid
  phone     String   @db.VarChar(15)
  codeHash  String   @map("code_hash") @db.VarChar(255)
  attempts  Int      @default(0)
  expiresAt DateTime @map("expires_at")
  createdAt DateTime @default(now()) @map("created_at")

  @@index([phone])
  @@map("otp_codes")
}

model RefreshToken {
  id        String    @id @default(uuid()) @db.Uuid
  userId    String    @map("user_id") @db.Uuid
  tokenHash String    @unique @map("token_hash") @db.VarChar(255)
  expiresAt DateTime  @map("expires_at")
  revokedAt DateTime? @map("revoked_at")
  createdAt DateTime  @default(now()) @map("created_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@map("refresh_tokens")
}

model PasswordResetToken {
  id        String    @id @default(uuid()) @db.Uuid
  userId    String    @map("user_id") @db.Uuid
  tokenHash String    @unique @map("token_hash") @db.VarChar(255)
  expiresAt DateTime  @map("expires_at")
  usedAt    DateTime? @map("used_at")
  createdAt DateTime  @default(now()) @map("created_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@map("password_reset_tokens")
}

model Address {
  id        String  @id @default(uuid()) @db.Uuid
  userId    String  @map("user_id") @db.Uuid
  label     String  @db.VarChar(50)
  fullName  String  @map("full_name") @db.VarChar(200)
  phone     String  @db.VarChar(15)
  line1     String  @db.VarChar(500)
  line2     String? @db.VarChar(500)
  city      String  @db.VarChar(100)
  state     String  @db.VarChar(100)
  pinCode   String  @map("pin_code") @db.VarChar(10)
  isDefault Boolean @default(false) @map("is_default")

  user   User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  orders Order[]

  @@map("addresses")
}

// ==========================================
// PET MARKETPLACE
// ==========================================

model PetSpecies {
  id       String  @id @default(uuid()) @db.Uuid
  name     String  @unique @db.VarChar(50)
  slug     String  @unique @db.VarChar(50)
  iconUrl  String? @map("icon_url") @db.VarChar(500)
  isActive Boolean @default(true) @map("is_active")

  breeds   PetBreed[]
  listings PetListing[]

  @@map("pet_species")
}

model PetBreed {
  id        String  @id @default(uuid()) @db.Uuid
  speciesId String  @map("species_id") @db.Uuid
  name      String  @db.VarChar(100)
  slug      String  @db.VarChar(100)
  isPopular Boolean @default(false) @map("is_popular")

  species  PetSpecies   @relation(fields: [speciesId], references: [id])
  listings PetListing[]

  @@unique([speciesId, slug])
  @@map("pet_breeds")
}

model PetListing {
  id              String           @id @default(uuid()) @db.Uuid
  sellerId        String           @map("seller_id") @db.Uuid
  speciesId       String           @map("species_id") @db.Uuid
  breedId         String           @map("breed_id") @db.Uuid
  title           String           @db.VarChar(200)
  description     String           @db.Text
  gender          Gender
  ageMonths       Int              @map("age_months")
  price           Decimal          @db.Decimal(10, 2)
  priceType       PriceType        @default(FIXED) @map("price_type")
  status          PetListingStatus @default(DRAFT)
  city            String           @db.VarChar(100)
  state           String           @db.VarChar(100)
  isVaccinated    Boolean          @default(false) @map("is_vaccinated")
  isNeutered      Boolean          @default(false) @map("is_neutered")
  viewCount       Int              @default(0) @map("view_count")
  isBoosted       Boolean          @default(false) @map("is_boosted")
  boostedUntil    DateTime?        @map("boosted_until")
  images          String[]         @default([])
  // Admin moderation (Phase 4b). isFlagged is separate from `status` --
  // flagging marks an already-ACTIVE listing for re-review without
  // removing it from public view, per the spec's 6.4 "Flag Listing".
  isFlagged       Boolean          @default(false) @map("is_flagged")
  flagReason      String?          @map("flag_reason") @db.VarChar(500)
  flaggedAt       DateTime?        @map("flagged_at")
  moderationNotes String?          @map("moderation_notes") @db.Text
  // One of the spec's fixed rejection reason codes (poor_photos,
  // incomplete_info, prohibited_species, suspected_fraud,
  // health_concerns, price_unrealistic, duplicate) -- plain string,
  // validated at the application layer, matching this codebase's
  // existing convention for open-ended-but-known-set string fields.
  rejectionReason String?          @map("rejection_reason") @db.VarChar(50)
  moderatedAt     DateTime?        @map("moderated_at")
  moderatedById   String?          @map("moderated_by_id") @db.Uuid
  createdAt       DateTime         @default(now()) @map("created_at")
  updatedAt       DateTime         @updatedAt @map("updated_at")

  seller        User           @relation(fields: [sellerId], references: [id])
  species       PetSpecies     @relation(fields: [speciesId], references: [id])
  breed         PetBreed       @relation(fields: [breedId], references: [id])
  orderItems    OrderItem[]
  reports       PetReport[]
  conversations Conversation[]
  moderatedBy   User?          @relation("ListingModerator", fields: [moderatedById], references: [id])

  @@index([status, city, speciesId])
  @@map("pet_listings")
}

model PetReport {
  id           String   @id @default(uuid()) @db.Uuid
  petListingId String   @map("pet_listing_id") @db.Uuid
  reporterId   String   @map("reporter_id") @db.Uuid
  reason       String   @db.VarChar(500)
  createdAt    DateTime @default(now()) @map("created_at")

  petListing PetListing @relation(fields: [petListingId], references: [id], onDelete: Cascade)

  @@unique([petListingId, reporterId])
  @@map("pet_reports")
}

// ==========================================
// PRODUCTS
// ==========================================

model ProductCategory {
  id        String  @id @default(uuid()) @db.Uuid
  name      String  @unique @db.VarChar(100)
  slug      String  @unique @db.VarChar(100)
  iconUrl   String? @map("icon_url") @db.VarChar(500)
  // Admin-managed display order (Phase 4b `PUT /admin/categories/reorder`).
  // Defaults to 0 for all pre-existing rows -- safe, since the public
  // category list previously had no persisted order at all (implicit
  // DB/insertion order), so this is a genuinely new capability, not a
  // change to existing behavior that could break anything.
  sortOrder Int     @default(0) @map("sort_order")
  isActive  Boolean @default(true) @map("is_active")

  products Product[]

  @@map("product_categories")
}

model Product {
  id            String   @id @default(uuid()) @db.Uuid
  categoryId    String   @map("category_id") @db.Uuid
  name          String   @db.VarChar(200)
  slug          String   @unique @db.VarChar(200)
  description   String   @db.Text
  brand         String?  @db.VarChar(100)
  price         Decimal  @db.Decimal(10, 2)
  originalPrice Decimal? @map("original_price") @db.Decimal(10, 2)
  stock         Int      @default(0)
  images        String[] @default([])
  rating        Float    @default(0)
  reviewCount   Int      @default(0) @map("review_count")
  isActive      Boolean  @default(true) @map("is_active")
  createdAt     DateTime @default(now()) @map("created_at")
  updatedAt     DateTime @updatedAt @map("updated_at")

  category   ProductCategory @relation(fields: [categoryId], references: [id])
  orderItems OrderItem[]
  reviews    Review[]
  cartItems  CartItem[]

  @@map("products")
}

// ==========================================
// CART
// ==========================================

model Cart {
  id        String   @id @default(uuid()) @db.Uuid
  userId    String   @unique @map("user_id") @db.Uuid
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  user  User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  items CartItem[]

  @@map("carts")
}

model CartItem {
  id        String   @id @default(uuid()) @db.Uuid
  cartId    String   @map("cart_id") @db.Uuid
  productId String   @map("product_id") @db.Uuid
  quantity  Int      @default(1)
  createdAt DateTime @default(now()) @map("created_at")

  cart    Cart    @relation(fields: [cartId], references: [id], onDelete: Cascade)
  product Product @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@unique([cartId, productId])
  @@map("cart_items")
}

// ==========================================
// ORDERS
// ==========================================

model Order {
  id                 String         @id @default(uuid()) @db.Uuid
  userId             String         @map("user_id") @db.Uuid
  addressId          String?        @map("address_id") @db.Uuid
  status             OrderStatus    @default(PENDING_PAYMENT)
  subtotal           Decimal        @db.Decimal(10, 2)
  shipping           Decimal        @default(0) @db.Decimal(10, 2)
  tax                Decimal        @default(0) @db.Decimal(10, 2)
  couponId           String?        @map("coupon_id") @db.Uuid
  couponCode         String?        @map("coupon_code") @db.VarChar(30)
  discountAmount     Decimal        @default(0) @map("discount_amount") @db.Decimal(10, 2)
  total              Decimal        @db.Decimal(10, 2)
  paymentMethod      PaymentMethod? @map("payment_method")
  paymentStatus      PaymentStatus  @default(CREATED) @map("payment_status")
  razorpayOrderId    String?        @map("razorpay_order_id") @db.VarChar(100)
  // Manually entered by the seller (or admin) -- no live courier API
  // integration exists (would need a real Shiprocket/Delhivery account,
  // same deferred-external-provider class as Razorpay). A real tracking
  // number a seller typed in is still real, useful data for the buyer.
  trackingNumber     String?        @map("tracking_number") @db.VarChar(100)
  carrier            String?        @db.VarChar(100)
  cancelReason       String?        @map("cancel_reason") @db.VarChar(500)
  cancelledAt        DateTime?      @map("cancelled_at")
  deliveredAt        DateTime?      @map("delivered_at")
  receiptConfirmedAt DateTime?      @map("receipt_confirmed_at")
  escrowStatus       EscrowStatus?  @map("escrow_status")
  escrowReleasedAt   DateTime?      @map("escrow_released_at")
  createdAt          DateTime       @default(now()) @map("created_at")
  updatedAt          DateTime       @updatedAt @map("updated_at")

  user     User        @relation(fields: [userId], references: [id])
  address  Address?    @relation(fields: [addressId], references: [id])
  coupon   Coupon?     @relation(fields: [couponId], references: [id])
  items    OrderItem[]
  payments Payment[]
  reviews  Review[]
  disputes Dispute[]

  @@map("orders")
}

enum CouponType {
  PERCENTAGE
  FLAT
}

// Simple, single-tier promo code engine (ORD-05 / ADM-09) -- no per-user
// redemption tracking or category/product-scoped coupons, since neither is
// in the frozen feature list's acceptance criteria ("Validate code, show
// discount, min order value, one coupon per order"). `usedCount` is
// incremented atomically (guarded update, same pattern as Product.stock
// decrement) at order-creation time to prevent a usage-limit race under
// concurrent checkouts.
model Coupon {
  id                String     @id @default(uuid()) @db.Uuid
  code              String     @unique @db.VarChar(30)
  type              CouponType
  // PERCENTAGE: 1-100. FLAT: a ₹ amount.
  value             Decimal    @db.Decimal(10, 2)
  minOrderValue     Decimal?   @map("min_order_value") @db.Decimal(10, 2)
  // Caps the discount for PERCENTAGE coupons; ignored for FLAT.
  maxDiscountAmount Decimal?   @map("max_discount_amount") @db.Decimal(10, 2)
  // Null = unlimited redemptions.
  usageLimit        Int?       @map("usage_limit")
  usedCount         Int        @default(0) @map("used_count")
  validFrom         DateTime   @default(now()) @map("valid_from")
  validUntil        DateTime?  @map("valid_until")
  isActive          Boolean    @default(true) @map("is_active")
  createdAt         DateTime   @default(now()) @map("created_at")

  orders Order[]

  @@map("coupons")
}

// Homepage banner management (ADM-12). `startDate`/`endDate` are both
// optional -- a banner with neither is simply "active whenever isActive is
// true", matching the spec's "schedule display dates" as an enhancement,
// not a requirement.
model Banner {
  id        String    @id @default(uuid()) @db.Uuid
  title     String    @db.VarChar(200)
  imageUrl  String    @map("image_url") @db.VarChar(500)
  linkUrl   String?   @map("link_url") @db.VarChar(500)
  sortOrder Int       @default(0) @map("sort_order")
  isActive  Boolean   @default(true) @map("is_active")
  startDate DateTime? @map("start_date")
  endDate   DateTime? @map("end_date")
  createdAt DateTime  @default(now()) @map("created_at")

  @@map("banners")
}

model OrderItem {
  id           String  @id @default(uuid()) @db.Uuid
  orderId      String  @map("order_id") @db.Uuid
  productId    String? @map("product_id") @db.Uuid
  petListingId String? @map("pet_listing_id") @db.Uuid
  name         String  @db.VarChar(200)
  price        Decimal @db.Decimal(10, 2)
  quantity     Int     @default(1)

  order         Order          @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product       Product?       @relation(fields: [productId], references: [id])
  petListing    PetListing?    @relation(fields: [petListingId], references: [id])
  returnRequest ReturnRequest?

  @@map("order_items")
}

enum ReturnStatus {
  REQUESTED
  APPROVED
  REJECTED
  COMPLETED
}

model ReturnRequest {
  id          String       @id @default(uuid()) @db.Uuid
  orderItemId String       @unique @map("order_item_id") @db.Uuid
  reason      String       @db.VarChar(100)
  description String       @db.Text
  photoUrls   String[]     @default([]) @map("photo_urls")
  status      ReturnStatus @default(REQUESTED)
  createdAt   DateTime     @default(now()) @map("created_at")

  orderItem OrderItem @relation(fields: [orderItemId], references: [id], onDelete: Cascade)

  @@map("return_requests")
}

// ==========================================
// PAYMENTS
// ==========================================

model Payment {
  id                String             @id @default(uuid()) @db.Uuid
  orderId           String             @map("order_id") @db.Uuid
  userId            String             @map("user_id") @db.Uuid
  razorpayOrderId   String?            @map("razorpay_order_id") @db.VarChar(100)
  razorpayPaymentId String?            @unique @map("razorpay_payment_id") @db.VarChar(100)
  amount            Decimal            @db.Decimal(10, 2)
  status            PaymentEventStatus @default(CREATED)
  refundedAmount    Decimal            @default(0) @map("refunded_amount") @db.Decimal(10, 2)
  createdAt         DateTime           @default(now()) @map("created_at")
  updatedAt         DateTime           @updatedAt @map("updated_at")

  order Order @relation(fields: [orderId], references: [id])
  user  User  @relation(fields: [userId], references: [id])

  @@index([orderId])
  @@index([userId])
  @@map("payments")
}

// Stores only a masked reference to the seller's bank account (last 4
// digits) — full account numbers should go through a PCI-scoped vault or
// Razorpay Route's own contact/fund-account storage in a real deployment,
// not this database.
model SellerBankAccount {
  id            String    @id @default(uuid()) @db.Uuid
  userId        String    @unique @map("user_id") @db.Uuid
  accountLast4  String    @map("account_last4") @db.VarChar(4)
  ifsc          String    @db.VarChar(11)
  accountHolder String    @map("account_holder") @db.VarChar(200)
  verifiedAt    DateTime? @map("verified_at")
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("seller_bank_accounts")
}

// ==========================================
// CHAT
// ==========================================

model Conversation {
  id            String   @id @default(uuid()) @db.Uuid
  petListingId  String   @map("pet_listing_id") @db.Uuid
  buyerId       String   @map("buyer_id") @db.Uuid
  sellerId      String   @map("seller_id") @db.Uuid
  blockedById   String?  @map("blocked_by_id") @db.Uuid
  blockedReason String?  @map("blocked_reason") @db.VarChar(500)
  createdAt     DateTime @default(now()) @map("created_at")

  petListing PetListing @relation(fields: [petListingId], references: [id], onDelete: Cascade)
  buyer      User       @relation("ConversationsAsBuyer", fields: [buyerId], references: [id])
  seller     User       @relation("ConversationsAsSeller", fields: [sellerId], references: [id])
  messages   Message[]

  @@unique([petListingId, buyerId])
  @@map("conversations")
}

model Message {
  id             String      @id @default(uuid()) @db.Uuid
  conversationId String      @map("conversation_id") @db.Uuid
  senderId       String      @map("sender_id") @db.Uuid
  content        String?     @db.Text
  messageType    MessageType @default(TEXT) @map("message_type")
  mediaUrl       String?     @map("media_url") @db.VarChar(500)
  isRead         Boolean     @default(false) @map("is_read")
  createdAt      DateTime    @default(now()) @map("created_at")

  conversation Conversation @relation(fields: [conversationId], references: [id], onDelete: Cascade)
  sender       User         @relation(fields: [senderId], references: [id])

  @@index([conversationId, createdAt])
  @@map("messages")
}

// ==========================================
// SERVICES
// ==========================================

// `status` gates public visibility: newly self-registered providers start
// PENDING and are invisible to the public list/detail endpoints until an
// ADMIN approves them. The column DEFAULTs to PENDING (fail-closed if any
// future code path forgets to set it explicitly) -- the migration for this
// change separately backfills all pre-existing (admin-seeded, already
// public) rows to APPROVED so they don't disappear from the public site.
// `isVerified` is a separate, admin-settable credential badge, orthogonal
// to whether the provider is listed at all.
model ServiceProvider {
  id              String                @id @default(uuid()) @db.Uuid
  ownerId         String?               @map("owner_id") @db.Uuid
  name            String                @db.VarChar(200)
  type            ServiceType
  description     String                @db.Text
  city            String                @db.VarChar(100)
  state           String                @db.VarChar(100)
  address         String?               @db.VarChar(500)
  phone           String?               @db.VarChar(15)
  priceFrom       Decimal               @map("price_from") @db.Decimal(10, 2)
  rating          Float                 @default(0)
  reviewCount     Int                   @default(0) @map("review_count")
  timing          String?               @db.VarChar(100)
  isActive        Boolean               @default(true) @map("is_active")
  status          ServiceProviderStatus @default(PENDING)
  isVerified      Boolean               @default(false) @map("is_verified")
  rejectionReason String?               @map("rejection_reason") @db.VarChar(500)
  // Plain lat/lng + an application-computed haversine distance for
  // "near me" search -- no PostGIS/spatial index in this pass (not
  // required at current scale; documented as a future enhancement, see
  // docs/phase-3b-services-completion-report.md).
  latitude        Float?
  longitude       Float?
  subTypes        String[]              @default([]) @map("sub_types")
  speciesServed   String[]              @default([]) @map("species_served")
  images          String[]              @default([])
  // Provider-type-specific registration details (license number,
  // qualification, experience, etc.) -- kept flexible since a vet's
  // credentials look nothing like a groomer's.
  credentials     Json?
  createdAt       DateTime              @default(now()) @map("created_at")

  owner    User?              @relation(fields: [ownerId], references: [id], onDelete: SetNull)
  services ProviderService[]
  schedule ProviderSchedule[]
  bookings Booking[]

  @@index([status, isActive, type])
  @@map("service_providers")
}

model ProviderService {
  id          String   @id @default(uuid()) @db.Uuid
  providerId  String   @map("provider_id") @db.Uuid
  name        String   @db.VarChar(200)
  description String?  @db.VarChar(1000)
  duration    Int // minutes
  price       Decimal  @db.Decimal(10, 2)
  isActive    Boolean  @default(true) @map("is_active")
  createdAt   DateTime @default(now()) @map("created_at")

  provider ServiceProvider @relation(fields: [providerId], references: [id], onDelete: Cascade)
  bookings Booking[]

  @@index([providerId])
  @@map("provider_services")
}

// A structured weekly schedule (replacing the old free-text `timing`
// field, which is left in place for backward display compatibility but no
// longer authoritative for availability computation). Availability is
// computed on read by generating candidate slots from this schedule minus
// overlapping bookings -- not persisted as individual slot rows, to avoid
// slot-row staleness/explosion.
model ProviderSchedule {
  id         String @id @default(uuid()) @db.Uuid
  providerId String @map("provider_id") @db.Uuid
  dayOfWeek  Int    @map("day_of_week") // 0 (Sunday) - 6 (Saturday)
  openTime   String @map("open_time") @db.VarChar(5) // "HH:mm"
  closeTime  String @map("close_time") @db.VarChar(5)

  provider ServiceProvider @relation(fields: [providerId], references: [id], onDelete: Cascade)

  @@unique([providerId, dayOfWeek])
  @@map("provider_schedules")
}

model Booking {
  id         String        @id @default(uuid()) @db.Uuid
  userId     String        @map("user_id") @db.Uuid
  providerId String        @map("provider_id") @db.Uuid
  // Nullable for migration safety against any pre-existing rows created by
  // the old minimal booking flow; every booking created going forward
  // always sets these.
  serviceId  String?       @map("service_id") @db.Uuid
  date       DateTime
  startTime  DateTime?     @map("start_time")
  endTime    DateTime?     @map("end_time")
  status     BookingStatus @default(PENDING)
  notes      String?       @db.VarChar(500)

  confirmationCode  String?   @map("confirmation_code") @db.VarChar(20)
  cancelReason      String?   @map("cancel_reason") @db.VarChar(500)
  cancelledAt       DateTime? @map("cancelled_at")
  // Computed by the cancellation notice-period policy (see the completion
  // report) -- informational only. No real Payment/Razorpay integration
  // exists for bookings yet (see below), so this never triggers an actual
  // refund transaction; it records what *would* be owed.
  refundAmount      Decimal?  @map("refund_amount") @db.Decimal(10, 2)
  rescheduledFromId String?   @map("rescheduled_from_id") @db.Uuid
  createdAt         DateTime  @default(now()) @map("created_at")

  user            User             @relation(fields: [userId], references: [id])
  provider        ServiceProvider  @relation(fields: [providerId], references: [id])
  service         ProviderService? @relation(fields: [serviceId], references: [id])
  rescheduledFrom Booking?         @relation("BookingReschedule", fields: [rescheduledFromId], references: [id])
  rescheduledTo   Booking[]        @relation("BookingReschedule")

  // A provider (modeled here as a single resource, not multi-staff) can
  // only hold one booking per exact start time -- DB-enforced so two
  // concurrent booking attempts for the same slot can never both succeed
  // (the second gets a real unique-constraint conflict, not a silent
  // overwrite). NULL start_time (legacy rows) is exempt, since Postgres
  // treats multiple NULLs as distinct.
  @@unique([providerId, startTime])
  @@index([providerId, startTime])
  @@index([userId])
  @@map("bookings")
}

// ==========================================
// REVIEWS
// ==========================================

enum ReviewReportReason {
  SPAM
  INAPPROPRIATE
  FAKE
  IRRELEVANT
  HARASSMENT
}

model Review {
  id                String    @id @default(uuid()) @db.Uuid
  userId            String    @map("user_id") @db.Uuid
  productId         String    @map("product_id") @db.Uuid
  orderId           String    @map("order_id") @db.Uuid
  title             String    @db.VarChar(100)
  rating            Int
  comment           String    @db.Text
  images            String[]  @default([])
  tags              String[]  @default([])
  helpfulCount      Int       @default(0) @map("helpful_count")
  sellerResponse    String?   @map("seller_response") @db.Text
  sellerRespondedAt DateTime? @map("seller_responded_at")
  editedAt          DateTime? @map("edited_at")
  deletedAt         DateTime? @map("deleted_at")
  createdAt         DateTime  @default(now()) @map("created_at")

  user         User                @relation(fields: [userId], references: [id])
  product      Product             @relation(fields: [productId], references: [id])
  order        Order               @relation(fields: [orderId], references: [id])
  helpfulVotes ReviewHelpfulVote[]
  reports      ReviewReport[]

  @@unique([orderId, productId, userId])
  @@index([productId])
  @@index([userId])
  @@map("reviews")
}

model ReviewHelpfulVote {
  id        String   @id @default(uuid()) @db.Uuid
  reviewId  String   @map("review_id") @db.Uuid
  userId    String   @map("user_id") @db.Uuid
  createdAt DateTime @default(now()) @map("created_at")

  review Review @relation(fields: [reviewId], references: [id], onDelete: Cascade)

  @@unique([reviewId, userId])
  @@map("review_helpful_votes")
}

model ReviewReport {
  id         String             @id @default(uuid()) @db.Uuid
  reviewId   String             @map("review_id") @db.Uuid
  reporterId String             @map("reporter_id") @db.Uuid
  reason     ReviewReportReason
  details    String?            @db.VarChar(500)
  createdAt  DateTime           @default(now()) @map("created_at")

  review Review @relation(fields: [reviewId], references: [id], onDelete: Cascade)

  @@unique([reviewId, reporterId])
  @@map("review_reports")
}

// ==========================================
// NOTIFICATIONS (Phase 3a)
// ==========================================
// `Notification.type` is a plain String, not a Prisma enum: the spec's own
// event catalogue (ORDER_CONFIRMED, NEW_MESSAGE, BOOKING_REMINDER, ...) is
// still growing across Phase 3b/3c/3d (bookings, community, education) and
// each addition would otherwise need its own migration. Validity is
// enforced at the application layer (zod) against the currently-known set
// instead. `message` deliberately matches the field name proposed in the
// approved implementation plan; the spec's example response calls the same
// concept `body` -- documented, low-risk naming deviation, not a functional
// difference.

enum DevicePlatform {
  IOS
  ANDROID
  WEB
}

enum NotificationChannel {
  PUSH
  EMAIL
  SMS
}

enum OutboxStatus {
  PENDING
  SENT
  FAILED
  SKIPPED
}

model Notification {
  id         String    @id @default(uuid()) @db.Uuid
  userId     String    @map("user_id") @db.Uuid
  type       String    @db.VarChar(50)
  title      String    @db.VarChar(200)
  message    String    @db.VarChar(2000)
  icon       String?   @db.VarChar(50)
  image      String?   @db.VarChar(500)
  actionType String?   @map("action_type") @db.VarChar(50)
  actionUrl  String?   @map("action_url") @db.VarChar(500)
  // Small, structured template-fill data only (e.g. { orderId, orderNumber }).
  // Must never contain secrets, tokens, or payment details -- enforced by
  // size/shape validation at the service layer, not by the column itself.
  metadata   Json?
  // Optional caller-supplied idempotency key (e.g. "ORDER_CONFIRMED:<orderId>").
  // Postgres/Prisma treat multiple NULLs as distinct for uniqueness, so this
  // is a no-op for callers that don't pass one, and a real DB-enforced
  // duplicate guard (not just a check-then-insert race) for callers that do.
  dedupeKey  String?   @map("dedupe_key") @db.VarChar(200)
  readAt     DateTime? @map("read_at")
  deletedAt  DateTime? @map("deleted_at")
  createdAt  DateTime  @default(now()) @map("created_at")

  user   User                 @relation(fields: [userId], references: [id], onDelete: Cascade)
  outbox NotificationOutbox[]

  @@unique([userId, dedupeKey])
  @@index([userId, createdAt])
  @@index([userId, readAt])
  @@index([userId, type])
  @@map("notifications")
}

model NotificationPreference {
  id                String   @id @default(uuid()) @db.Uuid
  userId            String   @unique @map("user_id") @db.Uuid
  // Nested toggle objects (enabled + per-category flags), matching the
  // spec's response shape directly -- see notifications.schema.ts for the
  // zod-validated shape and the application-level defaults (kept in code,
  // not as a schema-level JSON default literal, so they stay readable and
  // easy to extend with new categories without touching the DB default).
  // Nullable: a missing row/column value means "use the code-level
  // defaults", resolved by getOrCreatePreferences() before every read.
  push              Json?
  email             Json?
  sms               Json?
  quietHoursEnabled Boolean  @default(false) @map("quiet_hours_enabled")
  quietHoursFrom    String   @default("22:00") @map("quiet_hours_from") @db.VarChar(5)
  quietHoursTo      String   @default("07:00") @map("quiet_hours_to") @db.VarChar(5)
  createdAt         DateTime @default(now()) @map("created_at")
  updatedAt         DateTime @updatedAt @map("updated_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("notification_preferences")
}

model DeviceToken {
  id         String         @id @default(uuid()) @db.Uuid
  userId     String         @map("user_id") @db.Uuid
  token      String         @unique @db.VarChar(500)
  platform   DevicePlatform
  deviceId   String         @map("device_id") @db.VarChar(200)
  appVersion String?        @map("app_version") @db.VarChar(20)
  createdAt  DateTime       @default(now()) @map("created_at")
  updatedAt  DateTime       @updatedAt @map("updated_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([userId, deviceId])
  @@map("device_tokens")
}

model NotificationOutbox {
  id                String              @id @default(uuid()) @db.Uuid
  notificationId    String              @map("notification_id") @db.Uuid
  channel           NotificationChannel
  status            OutboxStatus        @default(PENDING)
  attemptCount      Int                 @default(0) @map("attempt_count")
  nextAttemptAt     DateTime?           @map("next_attempt_at")
  // Sanitized failure summary only -- never a raw provider response body,
  // which could contain the recipient's contact details or provider
  // internals. See notifications.service.ts for the sanitization point.
  lastError         String?             @map("last_error") @db.VarChar(500)
  providerMessageId String?             @map("provider_message_id") @db.VarChar(200)
  createdAt         DateTime            @default(now()) @map("created_at")
  processedAt       DateTime?           @map("processed_at")

  notification Notification @relation(fields: [notificationId], references: [id], onDelete: Cascade)

  @@index([notificationId])
  @@index([status, nextAttemptAt])
  @@map("notification_outbox")
}

// ==========================================
// COMMUNITY (Phase 3c)
// ==========================================
// `Post.category` IS a Prisma enum (unlike Notification.type) -- unlike the
// notification event catalogue, the forum category list is a small, stable
// taxonomy intrinsic to this domain, not something new Phase 3 modules will
// keep adding to.
//
// Vote/reply-count aggregates (upvotes/downvotes/netScore/replyCount) are
// recomputed from the real PostVote/Reply rows inside the same transaction
// as the triggering write, and the parent row (Post/Reply) is locked
// FIRST, before any child-row insert -- the exact ordering the Phase 2
// verification's deadlock finding requires (a child INSERT referencing a
// parent FK implicitly takes a FOR KEY SHARE lock on the parent; locking
// the parent for real *after* that would deadlock two concurrent writers).

enum PostCategory {
  BREED_DISCUSSION
  HEALTH_NUTRITION
  TRAINING_TIPS
  LOST_AND_FOUND
  BUY_SELL_ADVICE
  GENERAL
}

enum LostFoundType {
  LOST
  FOUND
}

model Post {
  id         String       @id @default(uuid()) @db.Uuid
  authorId   String       @map("author_id") @db.Uuid
  category   PostCategory
  title      String       @db.VarChar(200)
  body       String       @db.Text
  images     String[]     @default([])
  species    String?      @db.VarChar(50)
  breed      String?      @db.VarChar(100)
  tags       String[]     @default([])
  isPinned   Boolean      @default(false) @map("is_pinned")
  viewCount  Int          @default(0) @map("view_count")
  upvotes    Int          @default(0)
  downvotes  Int          @default(0)
  // Denormalized (upvotes - downvotes), recomputed alongside them --
  // stored so "most_voted" sort can use a plain orderBy instead of raw SQL.
  netScore   Int          @default(0) @map("net_score")
  replyCount Int          @default(0) @map("reply_count")
  editedAt   DateTime?    @map("edited_at")
  deletedAt  DateTime?    @map("deleted_at")
  createdAt  DateTime     @default(now()) @map("created_at")

  author  User         @relation(fields: [authorId], references: [id])
  votes   PostVote[]
  follows PostFollow[]
  replies Reply[]

  @@index([category, createdAt])
  @@index([deletedAt])
  @@map("community_posts")
}

model PostVote {
  id        String   @id @default(uuid()) @db.Uuid
  postId    String   @map("post_id") @db.Uuid
  userId    String   @map("user_id") @db.Uuid
  value     Int // 1 (upvote) or -1 (downvote); a 0 request deletes the row instead of storing it
  createdAt DateTime @default(now()) @map("created_at")

  post Post @relation(fields: [postId], references: [id], onDelete: Cascade)
  user User @relation(fields: [userId], references: [id])

  @@unique([postId, userId])
  @@map("community_post_votes")
}

model PostFollow {
  id        String   @id @default(uuid()) @db.Uuid
  postId    String   @map("post_id") @db.Uuid
  userId    String   @map("user_id") @db.Uuid
  createdAt DateTime @default(now()) @map("created_at")

  post Post @relation(fields: [postId], references: [id], onDelete: Cascade)
  user User @relation(fields: [userId], references: [id])

  @@unique([postId, userId])
  @@map("community_post_follows")
}

model Reply {
  id        String    @id @default(uuid()) @db.Uuid
  postId    String    @map("post_id") @db.Uuid
  authorId  String    @map("author_id") @db.Uuid
  // Self-relation for one level of nesting. "Max 2 levels" is enforced at
  // the application layer (reject creating a reply whose parent already
  // has a non-null parentId), not by the schema itself.
  parentId  String?   @map("parent_id") @db.Uuid
  body      String    @db.Text
  images    String[]  @default([])
  upvotes   Int       @default(0)
  downvotes Int       @default(0)
  netScore  Int       @default(0) @map("net_score")
  editedAt  DateTime? @map("edited_at")
  deletedAt DateTime? @map("deleted_at")
  createdAt DateTime  @default(now()) @map("created_at")

  post     Post        @relation(fields: [postId], references: [id], onDelete: Cascade)
  author   User        @relation(fields: [authorId], references: [id])
  parent   Reply?      @relation("ReplyNesting", fields: [parentId], references: [id])
  children Reply[]     @relation("ReplyNesting")
  votes    ReplyVote[]

  @@index([postId, createdAt])
  @@map("community_replies")
}

model ReplyVote {
  id        String   @id @default(uuid()) @db.Uuid
  replyId   String   @map("reply_id") @db.Uuid
  userId    String   @map("user_id") @db.Uuid
  value     Int
  createdAt DateTime @default(now()) @map("created_at")

  reply Reply @relation(fields: [replyId], references: [id], onDelete: Cascade)
  user  User  @relation(fields: [userId], references: [id])

  @@unique([replyId, userId])
  @@map("community_reply_votes")
}

// Geo-search (lat/lng/radius) reuses the same real-haversine-in-raw-SQL
// approach as ServiceProvider search from Phase 3b -- no PostGIS, per the
// approved decision. "Notify users within 5km" from the spec is NOT wired
// up this pass (would call into the Notifications module) -- deferred to
// the Phase 3 final-integration step, the same consistent deferral applied
// to every other cross-module event source so far (Notifications' own
// createNotification() isn't yet called by Orders/Chat/Reviews either).
model LostFoundPost {
  id           String        @id @default(uuid()) @db.Uuid
  reporterId   String        @map("reporter_id") @db.Uuid
  type         LostFoundType
  petName      String        @map("pet_name") @db.VarChar(100)
  species      String        @db.VarChar(50)
  breed        String?       @db.VarChar(100)
  color        String        @db.VarChar(200)
  description  String        @db.Text
  images       String[]      @default([])
  lastSeenAt   String        @map("last_seen_at") @db.VarChar(300)
  latitude     Float
  longitude    Float
  city         String        @db.VarChar(100)
  contactPhone String        @map("contact_phone") @db.VarChar(20)
  resolvedAt   DateTime?     @map("resolved_at")
  createdAt    DateTime      @default(now()) @map("created_at")

  reporter User @relation(fields: [reporterId], references: [id])

  @@index([type, resolvedAt, city])
  @@map("community_lost_found_posts")
}

// ==========================================
// EDUCATION (Phase 3d)
// ==========================================
// "Who counts as a vet / a verified service provider" is NOT a new Role
// enum value -- it reuses Phase 3b's ServiceProvider approval system
// exactly like the Community badge check does: a user is treated as a vet
// if they own an APPROVED ServiceProvider with type VET_CONSULTATION, and
// as a "verified service provider" (for content-creation rights) if they
// own ANY APPROVED ServiceProvider. This avoids a parallel verification
// system alongside the one that already exists and is already tested.
//
// Course/consultation payments deliberately do NOT link to the existing
// Payment model -- Payment.orderId is a mandatory FK to Order (confirmed
// in the Phase 3b Services report), so it cannot represent a course- or
// consultation-only payment without a schema change to Payment itself that
// wasn't decided. Premium course enrollment and paid consultations fail
// fast with the same PAYMENT_PROVIDER_UNAVAILABLE code Razorpay itself uses
// when unconfigured, rather than faking success or duplicating payment
// logic. `Course.price`/`VetConsultation.amount` are informational only.
//
// `videoRoomUrl` is left null until a real video provider exists (see
// education/video-room.provider.ts) -- never fabricated.

enum EducationContentType {
  ARTICLE
  VIDEO
  GUIDE
}

enum EducationDifficulty {
  BEGINNER
  INTERMEDIATE
  ADVANCED
}

model EducationContent {
  id           String               @id @default(uuid()) @db.Uuid
  type         EducationContentType
  title        String               @db.VarChar(200)
  slug         String               @unique @db.VarChar(200)
  body         String?              @db.Text
  thumbnailUrl String?              @map("thumbnail_url") @db.VarChar(500)
  videoUrl     String?              @map("video_url") @db.VarChar(500)
  duration     Int? // seconds, video content only
  category     String               @db.VarChar(50)
  species      String?              @db.VarChar(50)
  breed        String?              @db.VarChar(100)
  difficulty   EducationDifficulty
  authorId     String               @map("author_id") @db.Uuid
  viewCount    Int                  @default(0) @map("view_count")
  likeCount    Int                  @default(0) @map("like_count")
  isPremium    Boolean              @default(false) @map("is_premium")
  tags         String[]             @default([])
  createdAt    DateTime             @default(now()) @map("created_at")

  author      User            @relation(fields: [authorId], references: [id])
  likes       ContentLike[]
  courseLinks CourseContent[]

  @@index([category, createdAt])
  @@map("education_content")
}

model ContentLike {
  id        String   @id @default(uuid()) @db.Uuid
  contentId String   @map("content_id") @db.Uuid
  userId    String   @map("user_id") @db.Uuid
  createdAt DateTime @default(now()) @map("created_at")

  content EducationContent @relation(fields: [contentId], references: [id], onDelete: Cascade)
  user    User             @relation(fields: [userId], references: [id])

  @@unique([contentId, userId])
  @@map("education_content_likes")
}

model Course {
  id          String              @id @default(uuid()) @db.Uuid
  title       String              @db.VarChar(200)
  slug        String              @unique @db.VarChar(200)
  description String              @db.Text
  thumbnail   String?             @db.VarChar(500)
  category    String              @db.VarChar(50)
  species     String?             @db.VarChar(50)
  difficulty  EducationDifficulty
  isPremium   Boolean             @default(false) @map("is_premium")
  price       Decimal?            @db.Decimal(10, 2)
  authorId    String              @map("author_id") @db.Uuid
  createdAt   DateTime            @default(now()) @map("created_at")

  author      User               @relation(fields: [authorId], references: [id])
  contents    CourseContent[]
  enrollments CourseEnrollment[]
  progress    CourseProgress[]

  @@map("education_courses")
}

model CourseContent {
  id        String @id @default(uuid()) @db.Uuid
  courseId  String @map("course_id") @db.Uuid
  contentId String @map("content_id") @db.Uuid
  order     Int

  course  Course           @relation(fields: [courseId], references: [id], onDelete: Cascade)
  content EducationContent @relation(fields: [contentId], references: [id], onDelete: Cascade)

  @@unique([courseId, contentId])
  @@index([courseId, order])
  @@map("education_course_contents")
}

model CourseEnrollment {
  id         String   @id @default(uuid()) @db.Uuid
  courseId   String   @map("course_id") @db.Uuid
  userId     String   @map("user_id") @db.Uuid
  enrolledAt DateTime @default(now()) @map("enrolled_at")

  course Course @relation(fields: [courseId], references: [id], onDelete: Cascade)
  user   User   @relation(fields: [userId], references: [id])

  @@unique([courseId, userId])
  @@map("education_course_enrollments")
}

model CourseProgress {
  id          String   @id @default(uuid()) @db.Uuid
  courseId    String   @map("course_id") @db.Uuid
  userId      String   @map("user_id") @db.Uuid
  contentId   String   @map("content_id") @db.Uuid
  completedAt DateTime @default(now()) @map("completed_at")

  course Course @relation(fields: [courseId], references: [id], onDelete: Cascade)
  user   User   @relation(fields: [userId], references: [id])

  @@unique([courseId, userId, contentId])
  @@map("education_course_progress")
}

enum VetConsultationType {
  VIDEO
  CHAT
}

enum VetConsultationStatus {
  SCHEDULED
  COMPLETED
  CANCELLED
}

model VetConsultation {
  id           String                @id @default(uuid()) @db.Uuid
  userId       String                @map("user_id") @db.Uuid
  vetId        String                @map("vet_id") @db.Uuid
  type         VetConsultationType
  petName      String                @map("pet_name") @db.VarChar(100)
  petSpecies   String                @map("pet_species") @db.VarChar(50)
  petBreed     String?               @map("pet_breed") @db.VarChar(100)
  symptoms     String                @db.Text
  attachments  String[]              @default([])
  status       VetConsultationStatus @default(SCHEDULED)
  scheduledAt  DateTime              @map("scheduled_at")
  // Never fabricated -- null/pending until a real video provider exists.
  videoRoomUrl String?               @map("video_room_url") @db.VarChar(500)
  diagnosis    String?               @db.Text
  prescription Json?
  advice       String?               @db.Text
  followUpDays Int?                  @map("follow_up_days")
  // Informational only -- no real Payment link (see the section comment above).
  amount       Decimal               @db.Decimal(10, 2)
  createdAt    DateTime              @default(now()) @map("created_at")

  user User @relation("ConsultationsAsPatient", fields: [userId], references: [id])
  vet  User @relation("ConsultationsAsVet", fields: [vetId], references: [id])

  @@index([userId])
  @@index([vetId])
  @@map("vet_consultations")
}

model VetQA {
  id         String    @id @default(uuid()) @db.Uuid
  askerId    String    @map("asker_id") @db.Uuid
  question   String    @db.Text
  species    String?   @db.VarChar(50)
  breed      String?   @db.VarChar(100)
  images     String[]  @default([])
  answererId String?   @map("answerer_id") @db.Uuid
  answer     String?   @db.Text
  answeredAt DateTime? @map("answered_at")
  createdAt  DateTime  @default(now()) @map("created_at")

  asker    User  @relation("VetQaAsker", fields: [askerId], references: [id])
  answerer User? @relation("VetQaAnswerer", fields: [answererId], references: [id])

  @@index([createdAt])
  @@map("vet_qa")
}

// ==========================================
// PET INSURANCE (Phase 4a)
// ==========================================
//
// `InsurancePlan`/`InsurancePartner` were originally catalog-only (seeded,
// like ProductCategory/PetSpecies) -- the spec's Part 3 has no "create a
// plan" endpoint and Phase 4's Admin module didn't list one either. Admin
// management (`/admin/insurance/partners`, `/admin/insurance/plans`) was
// added later on explicit request, reusing these same models -- no schema
// change was needed, only new admin-only service/router code (see
// `admin-insurance.router.ts`).
//
// Policy purchase (`POST /insurance/policies`) is NOT gated behind a real
// payment the way premium course enrollment is -- it follows the Services
// Booking precedent instead: the policy is created for real immediately
// (status ACTIVE, a real generated policyNumber, computed start/end dates),
// with `premiumPaid` stored as informational data only. This is a
// deliberate, documented choice (not an oversight): insurance and services
// bookings are both "the core feature has real value the moment it's
// created" products, unlike premium courses/vet consultations, which are
// paid add-ons to an otherwise-free platform. The same underlying
// limitation applies either way -- `Payment.orderId` is a mandatory FK to
// `Order`, so no real `Payment` row can be linked to a policy without a
// schema change to `Payment` itself (the same open cross-phase decision
// raised in the Phase 3b/3d completion reports).

enum InsuranceCoverageType {
  BASIC
  COMPREHENSIVE
  ACCIDENT_ONLY
}

enum InsurancePaymentFrequency {
  MONTHLY
  YEARLY
}

enum InsurancePolicyStatus {
  ACTIVE
  CANCELLED
  EXPIRED
}

enum InsuranceClaimStatus {
  SUBMITTED
  UNDER_REVIEW
  APPROVED
  REJECTED
  PAID
}

model InsurancePartner {
  id       String  @id @default(uuid()) @db.Uuid
  name     String  @db.VarChar(200)
  logo     String? @db.VarChar(500)
  isActive Boolean @default(true) @map("is_active")

  plans InsurancePlan[]

  @@map("insurance_partners")
}

model InsurancePlan {
  id                String                @id @default(uuid()) @db.Uuid
  partnerId         String                @map("partner_id") @db.Uuid
  name              String                @db.VarChar(200)
  coverageType      InsuranceCoverageType @map("coverage_type")
  speciesCovered    String[]              @map("species_covered")
  coverageAmount    Decimal               @map("coverage_amount") @db.Decimal(12, 2)
  premiumMonthly    Decimal               @map("premium_monthly") @db.Decimal(10, 2)
  premiumYearly     Decimal               @map("premium_yearly") @db.Decimal(10, 2)
  deductible        Decimal               @db.Decimal(10, 2)
  waitingPeriodDays Int                   @map("waiting_period_days")
  // Nullable -- not every plan has an upper age limit.
  maxAgeMonths      Int?                  @map("max_age_months")
  // {accidents, illnesses, surgery, hospitalization, thirdPartyLiability, lostPet} booleans --
  // kept as Json (like ServiceProvider.credentials) rather than 6 boolean columns, since this
  // maps directly to the spec's nested "coverageDetails" object and different partners may add
  // more coverage flags over time without a migration.
  coverageDetails   Json                  @map("coverage_details")
  exclusions        String[]
  isActive          Boolean               @default(true) @map("is_active")
  createdAt         DateTime              @default(now()) @map("created_at")

  partner  InsurancePartner  @relation(fields: [partnerId], references: [id])
  policies InsurancePolicy[]

  @@index([coverageType])
  @@map("insurance_plans")
}

model InsurancePolicy {
  id               String                    @id @default(uuid()) @db.Uuid
  userId           String                    @map("user_id") @db.Uuid
  planId           String                    @map("plan_id") @db.Uuid
  policyNumber     String                    @unique @map("policy_number") @db.VarChar(50)
  petName          String                    @map("pet_name") @db.VarChar(100)
  petSpecies       String                    @map("pet_species") @db.VarChar(50)
  petBreed         String?                   @map("pet_breed") @db.VarChar(100)
  petAgeMonths     Int                       @map("pet_age_months")
  petGender        String                    @map("pet_gender") @db.VarChar(20)
  paymentFrequency InsurancePaymentFrequency @map("payment_frequency")
  // Informational only -- see the section comment above.
  premiumPaid      Decimal                   @map("premium_paid") @db.Decimal(10, 2)
  status           InsurancePolicyStatus     @default(ACTIVE)
  startDate        DateTime                  @map("start_date")
  endDate          DateTime                  @map("end_date")
  documentUrl      String?                   @map("document_url") @db.VarChar(500)
  cancelReason     String?                   @map("cancel_reason") @db.VarChar(500)
  cancelledAt      DateTime?                 @map("cancelled_at")
  refundAmount     Decimal?                  @map("refund_amount") @db.Decimal(10, 2)
  createdAt        DateTime                  @default(now()) @map("created_at")

  user   User             @relation(fields: [userId], references: [id])
  plan   InsurancePlan    @relation(fields: [planId], references: [id])
  claims InsuranceClaim[]

  @@index([userId])
  @@map("insurance_policies")
}

model InsuranceClaim {
  id           String               @id @default(uuid()) @db.Uuid
  policyId     String               @map("policy_id") @db.Uuid
  claimNumber  String               @unique @map("claim_number") @db.VarChar(50)
  incidentDate DateTime             @map("incident_date")
  description  String               @db.Text
  claimAmount  Decimal              @map("claim_amount") @db.Decimal(10, 2)
  documents    String[]
  status       InsuranceClaimStatus @default(SUBMITTED)
  createdAt    DateTime             @default(now()) @map("created_at")

  policy InsurancePolicy @relation(fields: [policyId], references: [id])

  // "Claim already filed for this incident" (DUPLICATE_CLAIM, 409) -- one
  // claim per (policy, incidentDate) rather than a free-for-all.
  @@unique([policyId, incidentDate])
  @@index([policyId])
  @@map("insurance_claims")
}

// ==========================================
// ADMIN (Phase 4b)
// ==========================================
//
// The spec (admin-api.md §2) describes 5 fine-grained admin sub-roles
// (super_admin/admin/moderator/support/finance) plus a TOTP-based MFA
// header for "sensitive operations". Neither exists anywhere in this
// codebase today (the `Role` enum only has BUYER/SELLER/BREEDER/ADMIN,
// and there is no TOTP/MFA library installed). Building a full RBAC
// tier system and real MFA are separate, larger architectural decisions
// -- out of scope for "implement the Admin API using existing auth
// infrastructure". Every admin endpoint in this module uses the
// existing `authorize("ADMIN")` middleware only, and no endpoint
// pretends to check an MFA header without a real TOTP implementation
// behind it (same "never fabricate" principle as Razorpay/video-rooms/
// document URLs elsewhere in this project). Documented here, not
// silently assumed.

enum KycType {
  SELLER
  BREEDER
  PROVIDER
}

enum KycStatus {
  PENDING
  APPROVED
  REJECTED
}

// No "submit KYC documents" endpoint exists anywhere in the Phase 1-3
// spec set (auth-api.md, users-api.md) -- only this admin-side review
// flow is documented. Community's badges.util.ts already flagged
// "verified_breeder needs a real KYC field that doesn't exist on User"
// as a known gap (Phase 3c). A minimal, real user-facing submission
// endpoint (POST /users/kyc) is added alongside this model so the admin
// review queue is actually reachable end-to-end rather than being
// permanently empty -- a small, necessary addition beyond the admin
// spec's literal scope, same reasoning as Education's added vet-qa
// list/respond endpoints in Phase 3d.
model KycSubmission {
  id              String    @id @default(uuid()) @db.Uuid
  userId          String    @map("user_id") @db.Uuid
  type            KycType
  documents       String[]
  status          KycStatus @default(PENDING)
  reviewNotes     String?   @map("review_notes") @db.Text
  rejectionReason String?   @map("rejection_reason") @db.VarChar(500)
  resubmitAllowed Boolean?  @map("resubmit_allowed")
  reviewedById    String?   @map("reviewed_by_id") @db.Uuid
  reviewedAt      DateTime? @map("reviewed_at")
  createdAt       DateTime  @default(now()) @map("created_at")

  user       User  @relation("KycSubmitter", fields: [userId], references: [id])
  reviewedBy User? @relation("KycReviewer", fields: [reviewedById], references: [id])

  @@index([status, type])
  @@map("kyc_submissions")
}

enum DisputeStatus {
  OPEN
  UNDER_REVIEW
  RESOLVED
}

enum DisputeResolution {
  FAVOR_BUYER
  FAVOR_SELLER
  PARTIAL_REFUND
  MUTUAL_AGREEMENT
}

model Dispute {
  id               String             @id @default(uuid()) @db.Uuid
  orderId          String             @map("order_id") @db.Uuid
  raisedById       String             @map("raised_by_id") @db.Uuid
  reason           String             @db.Text
  evidence         String[]           @default([])
  status           DisputeStatus      @default(OPEN)
  resolution       DisputeResolution?
  resolutionAction String?            @map("resolution_action") @db.VarChar(50)
  resolutionAmount Decimal?           @map("resolution_amount") @db.Decimal(10, 2)
  sellerPenalty    String?            @map("seller_penalty") @db.VarChar(50)
  adminNotes       String?            @map("admin_notes") @db.Text
  resolvedById     String?            @map("resolved_by_id") @db.Uuid
  resolvedAt       DateTime?          @map("resolved_at")
  createdAt        DateTime           @default(now()) @map("created_at")

  order      Order @relation(fields: [orderId], references: [id])
  raisedBy   User  @relation("DisputesRaised", fields: [raisedById], references: [id])
  resolvedBy User? @relation("DisputesResolved", fields: [resolvedById], references: [id])

  @@index([status])
  @@map("disputes")
}

enum PayoutStatus {
  PENDING
  PROCESSING
  COMPLETED
  FAILED
}

// Represents a batch payout to a seller for a given period -- computed
// from CAPTURED payments on that seller's PetListing orders within the
// period (Products have no seller -- see reviews.router.ts's precedent
// -- so only pet-sale orders currently contribute to a real payout
// amount).
model Payout {
  id          String       @id @default(uuid()) @db.Uuid
  sellerId    String       @map("seller_id") @db.Uuid
  amount      Decimal      @db.Decimal(10, 2)
  status      PayoutStatus @default(PENDING)
  periodStart DateTime     @map("period_start")
  periodEnd   DateTime     @map("period_end")
  processedAt DateTime?    @map("processed_at")
  createdAt   DateTime     @default(now()) @map("created_at")

  seller User @relation("SellerPayouts", fields: [sellerId], references: [id])

  @@index([sellerId])
  @@map("payouts")
}

// Singleton row -- exactly one, enforced at the application layer (a
// well-known fixed id, upserted rather than freely created).
model PlatformSettings {
  id                           String   @id @default(uuid())
  commissionPetSalePercent     Decimal  @default(5) @map("commission_pet_sale_percent") @db.Decimal(5, 2)
  commissionProductSalePercent Decimal  @default(15) @map("commission_product_sale_percent") @db.Decimal(5, 2)
  commissionServiceFeePercent  Decimal  @default(10) @map("commission_service_fee_percent") @db.Decimal(5, 2)
  autoApproveVerifiedBreeders  Boolean  @default(false) @map("auto_approve_verified_breeders")
  requirePhotoMin              Int      @default(3) @map("require_photo_min")
  maxListingsPerSeller         Int      @default(50) @map("max_listings_per_seller")
  freeDeliveryThreshold        Decimal  @default(999) @map("free_delivery_threshold") @db.Decimal(10, 2)
  defaultShippingFee           Decimal  @default(79) @map("default_shipping_fee") @db.Decimal(10, 2)
  updatedAt                    DateTime @updatedAt @map("updated_at")
  updatedById                  String?  @map("updated_by_id") @db.Uuid

  updatedBy User? @relation("SettingsUpdater", fields: [updatedById], references: [id])

  @@map("platform_settings")
}

model AuditLog {
  id         String   @id @default(uuid()) @db.Uuid
  adminId    String   @map("admin_id") @db.Uuid
  action     String   @db.VarChar(100)
  targetType String   @map("target_type") @db.VarChar(50)
  targetId   String   @map("target_id") @db.Uuid
  details    Json?
  ip         String?  @db.VarChar(50)
  createdAt  DateTime @default(now()) @map("created_at")

  admin User @relation("AuditActor", fields: [adminId], references: [id])

  @@index([adminId])
  @@index([targetType, targetId])
  @@index([createdAt])
  @@map("audit_logs")
}

// Public, unauthenticated footer newsletter signup -- deliberately not tied
// to a User account (a visitor can subscribe without registering).
model NewsletterSubscriber {
  id        String   @id @default(uuid()) @db.Uuid
  email     String   @unique @db.VarChar(255)
  createdAt DateTime @default(now()) @map("created_at")

  @@map("newsletter_subscribers")
}


```
