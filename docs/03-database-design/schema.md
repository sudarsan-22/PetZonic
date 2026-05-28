# PetZonic — Database Schema

> **Version**: 1.0.0  
> **Date**: May 28, 2026  
> **ORM**: Prisma 6 (PostgreSQL 16)

---

## Prisma Schema

```prisma
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
  BROKER
  FRANCHISE_OWNER
  VET
  GROOMER
  PET_SITTER
  PET_WALKER
  ADMIN
  SUPER_ADMIN
}

enum Gender {
  MALE
  FEMALE
}

enum KycDocumentType {
  AADHAAR
  PAN
  DRIVING_LICENSE
  PASSPORT
}

enum KycStatus {
  PENDING
  UNDER_REVIEW
  APPROVED
  REJECTED
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

enum MediaType {
  IMAGE
  VIDEO
}

enum OrderType {
  PRODUCT
  PET
  SERVICE
}

enum OrderStatus {
  PENDING_PAYMENT
  CONFIRMED
  PROCESSING
  PACKED
  SHIPPED
  OUT_FOR_DELIVERY
  DELIVERED
  CANCELLED
  RETURNED
  REFUNDED
}

enum OrderItemStatus {
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
  RETURNED
}

enum PaymentStatus {
  CREATED
  AUTHORIZED
  CAPTURED
  FAILED
  REFUNDED
  PARTIALLY_REFUNDED
}

enum PaymentMethod {
  UPI
  CREDIT_CARD
  DEBIT_CARD
  NET_BANKING
  WALLET
  COD
}

enum EscrowStatus {
  HELD
  RELEASED
  REFUNDED
  DISPUTED
}

enum PayoutStatus {
  PENDING
  PROCESSING
  COMPLETED
  FAILED
}

enum MessageType {
  TEXT
  IMAGE
  LOCATION
  SYSTEM
}

enum ServiceType {
  VET_CONSULTATION
  VET_HOME_VISIT
  GROOMING
  PET_SITTING
  PET_WALKING
  PET_BOARDING
  PET_TRAINING
}

enum BookingStatus {
  PENDING
  CONFIRMED
  IN_PROGRESS
  COMPLETED
  CANCELLED
  NO_SHOW
}

enum ReviewTargetType {
  SELLER
  BREEDER
  PRODUCT
  SERVICE_PROVIDER
  PET_LISTING
}

enum NotificationType {
  ORDER
  CHAT
  LISTING
  PAYMENT
  REVIEW
  SYSTEM
  PROMOTION
  REMINDER
}

enum VerificationStatus {
  UNVERIFIED
  PENDING
  VERIFIED
  EXPIRED
}

enum FranchiseStatus {
  APPLIED
  APPROVED
  ACTIVE
  SUSPENDED
  TERMINATED
}

// ==========================================
// USER DOMAIN
// ==========================================

model User {
  id            String      @id @default(uuid()) @db.Uuid
  phone         String?     @unique @db.VarChar(15)
  email         String?     @unique @db.VarChar(255)
  passwordHash  String?     @map("password_hash") @db.VarChar(255)
  status        UserStatus  @default(ACTIVE)
  emailVerified Boolean     @default(false) @map("email_verified")
  phoneVerified Boolean     @default(false) @map("phone_verified")
  lastLoginAt   DateTime?   @map("last_login_at")
  createdAt     DateTime    @default(now()) @map("created_at")
  updatedAt     DateTime    @updatedAt @map("updated_at")

  // Relations
  profile         UserProfile?
  roles           UserRole[]
  devices         UserDevice[]
  kycVerifications KycVerification[]
  breederProfile  BreederProfile?
  petListings     PetListing[]
  orders          Order[]          @relation("BuyerOrders")
  addresses       Address[]
  cart            Cart?
  payments        Payment[]
  reviewsWritten  Review[]         @relation("Reviewer")
  notifications   Notification[]
  chatRoomsAsBuyer  ChatRoom[]     @relation("BuyerChats")
  chatRoomsAsSeller ChatRoom[]     @relation("SellerChats")
  messagesSent    Message[]
  sellerPayouts   SellerPayout[]
  serviceProvider ServiceProvider?
  franchises      Franchise[]
  bookingsAsCustomer Booking[]     @relation("CustomerBookings")

  @@map("users")
}

model UserProfile {
  id          String   @id @default(uuid()) @db.Uuid
  userId      String   @unique @map("user_id") @db.Uuid
  firstName   String   @map("first_name") @db.VarChar(100)
  lastName    String?  @map("last_name") @db.VarChar(100)
  avatarUrl   String?  @map("avatar_url") @db.VarChar(500)
  bio         String?  @db.VarChar(500)
  city        String?  @db.VarChar(100)
  state       String?  @db.VarChar(100)
  latitude    Float?
  longitude   Float?
  gender      Gender?
  dateOfBirth Date?    @map("date_of_birth")
  preferences Json?    @db.JsonB

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("user_profiles")
}

model UserRole {
  id         String   @id @default(uuid()) @db.Uuid
  userId     String   @map("user_id") @db.Uuid
  role       Role
  isActive   Boolean  @default(true) @map("is_active")
  assignedAt DateTime @default(now()) @map("assigned_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([userId, role])
  @@map("user_roles")
}

model UserDevice {
  id         String   @id @default(uuid()) @db.Uuid
  userId     String   @map("user_id") @db.Uuid
  fcmToken   String   @map("fcm_token") @db.VarChar(500)
  deviceType String   @map("device_type") @db.VarChar(20)
  deviceName String?  @map("device_name") @db.VarChar(100)
  lastActive DateTime @default(now()) @map("last_active")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("user_devices")
}

model KycVerification {
  id                    String          @id @default(uuid()) @db.Uuid
  userId                String          @map("user_id") @db.Uuid
  documentType          KycDocumentType @map("document_type")
  documentNumberEncrypted String        @map("document_number_encrypted") @db.VarChar(500)
  documentUrl           String          @map("document_url") @db.VarChar(500)
  selfieUrl             String?         @map("selfie_url") @db.VarChar(500)
  status                KycStatus       @default(PENDING)
  rejectionReason       String?         @map("rejection_reason") @db.VarChar(500)
  verifiedBy            String?         @map("verified_by") @db.Uuid
  submittedAt           DateTime        @default(now()) @map("submitted_at")
  verifiedAt            DateTime?       @map("verified_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("kyc_verifications")
}

// ==========================================
// PET MARKETPLACE DOMAIN
// ==========================================

model PetSpecies {
  id        String @id @default(uuid()) @db.Uuid
  name      String @unique @db.VarChar(50)
  slug      String @unique @db.VarChar(50)
  iconUrl   String? @map("icon_url") @db.VarChar(500)
  sortOrder Int    @default(0) @map("sort_order")
  isActive  Boolean @default(true) @map("is_active")

  breeds   PetBreed[]
  listings PetListing[]

  @@map("pet_species")
}

model PetBreed {
  id          String  @id @default(uuid()) @db.Uuid
  speciesId   String  @map("species_id") @db.Uuid
  name        String  @db.VarChar(100)
  slug        String  @db.VarChar(100)
  description String? @db.VarChar(500)
  imageUrl    String? @map("image_url") @db.VarChar(500)
  isPopular   Boolean @default(false) @map("is_popular")

  species  PetSpecies   @relation(fields: [speciesId], references: [id])
  listings PetListing[]
  breederParents BreederParent[]

  @@unique([speciesId, slug])
  @@map("pet_breeds")
}

model PetListing {
  id           String           @id @default(uuid()) @db.Uuid
  sellerId     String           @map("seller_id") @db.Uuid
  speciesId    String           @map("species_id") @db.Uuid
  breedId      String           @map("breed_id") @db.Uuid
  title        String           @db.VarChar(200)
  description  String           @db.Text
  gender       Gender
  ageMonths    Int              @map("age_months")
  weightKg     Float?           @map("weight_kg")
  color        String?          @db.VarChar(50)
  price        Decimal          @db.Decimal(10, 2)
  priceType    PriceType        @default(FIXED) @map("price_type")
  status       PetListingStatus @default(DRAFT)
  city         String           @db.VarChar(100)
  state        String           @db.VarChar(100)
  latitude     Float?
  longitude    Float?
  isVaccinated Boolean          @default(false) @map("is_vaccinated")
  isNeutered   Boolean          @default(false) @map("is_neutered")
  healthInfo   Json?            @map("health_info") @db.JsonB
  viewCount    Int              @default(0) @map("view_count")
  isBoosted    Boolean          @default(false) @map("is_boosted")
  boostExpiresAt DateTime?      @map("boost_expires_at")
  expiresAt    DateTime         @map("expires_at")
  createdAt    DateTime         @default(now()) @map("created_at")
  updatedAt    DateTime         @updatedAt @map("updated_at")

  // Relations
  seller       User              @relation(fields: [sellerId], references: [id])
  species      PetSpecies        @relation(fields: [speciesId], references: [id])
  breed        PetBreed          @relation(fields: [breedId], references: [id])
  media        PetMedia[]
  vaccinations PetVaccination[]
  chatRooms    ChatRoom[]
  orderItems   OrderItem[]

  @@index([sellerId])
  @@index([status, city, speciesId])
  @@index([status, expiresAt])
  @@map("pet_listings")
}

model PetMedia {
  id           String    @id @default(uuid()) @db.Uuid
  petListingId String    @map("pet_listing_id") @db.Uuid
  url          String    @db.VarChar(500)
  mediaType    MediaType @default(IMAGE) @map("media_type")
  sortOrder    Int       @default(0) @map("sort_order")
  isPrimary    Boolean   @default(false) @map("is_primary")

  petListing PetListing @relation(fields: [petListingId], references: [id], onDelete: Cascade)

  @@map("pet_media")
}

model PetVaccination {
  id               String   @id @default(uuid()) @db.Uuid
  petListingId     String   @map("pet_listing_id") @db.Uuid
  vaccineName      String   @map("vaccine_name") @db.VarChar(100)
  administeredDate Date?    @map("administered_date")
  certificateUrl   String?  @map("certificate_url") @db.VarChar(500)

  petListing PetListing @relation(fields: [petListingId], references: [id], onDelete: Cascade)

  @@map("pet_vaccinations")
}

// ==========================================
// BREEDER DOMAIN
// ==========================================

model BreederProfile {
  id                 String             @id @default(uuid()) @db.Uuid
  userId             String             @unique @map("user_id") @db.Uuid
  kennelName         String?            @map("kennel_name") @db.VarChar(200)
  licenseNumber      String?            @map("license_number") @db.VarChar(50)
  licenseUrl         String?            @map("license_url") @db.VarChar(500)
  licenseExpiry      Date?              @map("license_expiry")
  experienceYears    Int?               @map("experience_years")
  breedingPhilosophy String?            @map("breeding_philosophy") @db.Text
  facilityPhotos     Json?              @map("facility_photos") @db.JsonB
  verificationStatus VerificationStatus @default(UNVERIFIED) @map("verification_status")
  trustScore         Float?             @map("trust_score")
  createdAt          DateTime           @default(now()) @map("created_at")

  user    User            @relation(fields: [userId], references: [id])
  parents BreederParent[]
  litters Litter[]

  @@map("breeder_profiles")
}

model BreederParent {
  id          String  @id @default(uuid()) @db.Uuid
  breederId   String  @map("breeder_id") @db.Uuid
  breedId     String  @map("breed_id") @db.Uuid
  name        String  @db.VarChar(100)
  gender      Gender
  birthDate   Date?   @map("birth_date")
  healthTests Json?   @map("health_tests") @db.JsonB
  photoUrl    String? @map("photo_url") @db.VarChar(500)
  pedigreeInfo String? @map("pedigree_info") @db.Text

  breeder  BreederProfile @relation(fields: [breederId], references: [id], onDelete: Cascade)
  breed    PetBreed       @relation(fields: [breedId], references: [id])
  littersAsSire Litter[]  @relation("Sire")
  littersAsDam  Litter[]  @relation("Dam")

  @@map("breeder_parents")
}

model Litter {
  id             String   @id @default(uuid()) @db.Uuid
  breederId      String   @map("breeder_id") @db.Uuid
  sireId         String   @map("sire_id") @db.Uuid
  damId          String   @map("dam_id") @db.Uuid
  birthDate      Date     @map("birth_date")
  totalCount     Int      @map("total_count")
  availableCount Int      @map("available_count")
  createdAt      DateTime @default(now()) @map("created_at")

  breeder BreederProfile @relation(fields: [breederId], references: [id])
  sire    BreederParent  @relation("Sire", fields: [sireId], references: [id])
  dam     BreederParent  @relation("Dam", fields: [damId], references: [id])

  @@map("litters")
}

// ==========================================
// PRODUCT STORE DOMAIN
// ==========================================

model ProductCategory {
  id        String  @id @default(uuid()) @db.Uuid
  parentId  String? @map("parent_id") @db.Uuid
  name      String  @db.VarChar(100)
  slug      String  @unique @db.VarChar(100)
  iconUrl   String? @map("icon_url") @db.VarChar(500)
  sortOrder Int     @default(0) @map("sort_order")
  isActive  Boolean @default(true) @map("is_active")

  parent   ProductCategory?  @relation("CategoryTree", fields: [parentId], references: [id])
  children ProductCategory[] @relation("CategoryTree")
  products Product[]

  @@map("product_categories")
}

model Product {
  id          String  @id @default(uuid()) @db.Uuid
  name        String  @db.VarChar(200)
  slug        String  @unique @db.VarChar(200)
  description String  @db.Text
  categoryId  String  @map("category_id") @db.Uuid
  brand       String? @db.VarChar(100)
  specifications Json? @db.JsonB
  isActive    Boolean @default(true) @map("is_active")
  basePrice   Decimal @map("base_price") @db.Decimal(10, 2)
  avgRating   Float   @default(0) @map("avg_rating")
  reviewCount Int     @default(0) @map("review_count")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  category ProductCategory  @relation(fields: [categoryId], references: [id])
  variants ProductVariant[]
  images   ProductImage[]

  @@index([categoryId, isActive])
  @@map("products")
}

model ProductVariant {
  id            String  @id @default(uuid()) @db.Uuid
  productId     String  @map("product_id") @db.Uuid
  name          String  @db.VarChar(100)
  sku           String  @unique @db.VarChar(50)
  price         Decimal @db.Decimal(10, 2)
  comparePrice  Decimal? @map("compare_price") @db.Decimal(10, 2)
  stockQuantity Int     @default(0) @map("stock_quantity")
  attributes    Json?   @db.JsonB
  imageUrl      String? @map("image_url") @db.VarChar(500)
  isActive      Boolean @default(true) @map("is_active")

  product   Product    @relation(fields: [productId], references: [id], onDelete: Cascade)
  cartItems CartItem[]
  orderItems OrderItem[]

  @@map("product_variants")
}

model ProductImage {
  id        String  @id @default(uuid()) @db.Uuid
  productId String  @map("product_id") @db.Uuid
  url       String  @db.VarChar(500)
  sortOrder Int     @default(0) @map("sort_order")
  isPrimary Boolean @default(false) @map("is_primary")

  product Product @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@map("product_images")
}

// ==========================================
// ORDER DOMAIN
// ==========================================

model Cart {
  id        String   @id @default(uuid()) @db.Uuid
  userId    String   @unique @map("user_id") @db.Uuid
  updatedAt DateTime @updatedAt @map("updated_at")

  user  User       @relation(fields: [userId], references: [id])
  items CartItem[]

  @@map("carts")
}

model CartItem {
  id               String @id @default(uuid()) @db.Uuid
  cartId           String @map("cart_id") @db.Uuid
  productVariantId String @map("product_variant_id") @db.Uuid
  quantity         Int    @default(1)

  cart           Cart           @relation(fields: [cartId], references: [id], onDelete: Cascade)
  productVariant ProductVariant @relation(fields: [productVariantId], references: [id])

  @@unique([cartId, productVariantId])
  @@map("cart_items")
}

model Order {
  id          String      @id @default(uuid()) @db.Uuid
  orderNumber String      @unique @map("order_number") @db.VarChar(20)
  buyerId     String      @map("buyer_id") @db.Uuid
  addressId   String?     @map("address_id") @db.Uuid
  orderType   OrderType   @map("order_type")
  status      OrderStatus @default(PENDING_PAYMENT)
  subtotal    Decimal     @db.Decimal(10, 2)
  shippingFee Decimal     @default(0) @map("shipping_fee") @db.Decimal(10, 2)
  discount    Decimal     @default(0) @db.Decimal(10, 2)
  tax         Decimal     @default(0) @db.Decimal(10, 2)
  total       Decimal     @db.Decimal(10, 2)
  couponCode  String?     @map("coupon_code") @db.VarChar(50)
  notes       String?     @db.VarChar(500)
  placedAt    DateTime    @default(now()) @map("placed_at")
  deliveredAt DateTime?   @map("delivered_at")
  cancelledAt DateTime?   @map("cancelled_at")

  buyer    User        @relation("BuyerOrders", fields: [buyerId], references: [id])
  address  Address?    @relation(fields: [addressId], references: [id])
  items    OrderItem[]
  payments Payment[]

  @@index([buyerId, status])
  @@index([orderNumber])
  @@map("orders")
}

model OrderItem {
  id               String          @id @default(uuid()) @db.Uuid
  orderId          String          @map("order_id") @db.Uuid
  productVariantId String?         @map("product_variant_id") @db.Uuid
  petListingId     String?         @map("pet_listing_id") @db.Uuid
  sellerId         String?         @map("seller_id") @db.Uuid
  quantity         Int             @default(1)
  unitPrice        Decimal         @map("unit_price") @db.Decimal(10, 2)
  totalPrice       Decimal         @map("total_price") @db.Decimal(10, 2)
  status           OrderItemStatus @default(PENDING)
  trackingNumber   String?         @map("tracking_number") @db.VarChar(100)
  trackingUrl      String?         @map("tracking_url") @db.VarChar(500)

  order          Order           @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productVariant ProductVariant? @relation(fields: [productVariantId], references: [id])
  petListing     PetListing?     @relation(fields: [petListingId], references: [id])
  escrowHolds    EscrowHold[]

  @@index([sellerId, status])
  @@map("order_items")
}

model Address {
  id           String  @id @default(uuid()) @db.Uuid
  userId       String  @map("user_id") @db.Uuid
  label        String? @db.VarChar(50)
  fullName     String  @map("full_name") @db.VarChar(100)
  phone        String  @db.VarChar(15)
  addressLine1 String  @map("address_line1") @db.VarChar(200)
  addressLine2 String? @map("address_line2") @db.VarChar(200)
  city         String  @db.VarChar(100)
  state        String  @db.VarChar(100)
  pincode      String  @db.VarChar(6)
  latitude     Float?
  longitude    Float?
  isDefault    Boolean @default(false) @map("is_default")

  user   User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  orders Order[]

  @@map("addresses")
}

// ==========================================
// PAYMENT DOMAIN
// ==========================================

model Payment {
  id                 String        @id @default(uuid()) @db.Uuid
  orderId            String        @map("order_id") @db.Uuid
  userId             String        @map("user_id") @db.Uuid
  razorpayOrderId    String?       @map("razorpay_order_id") @db.VarChar(100)
  razorpayPaymentId  String?       @unique @map("razorpay_payment_id") @db.VarChar(100)
  amount             Decimal       @db.Decimal(10, 2)
  status             PaymentStatus @default(CREATED)
  method             PaymentMethod?
  metadata           Json?         @db.JsonB
  createdAt          DateTime      @default(now()) @map("created_at")
  updatedAt          DateTime      @updatedAt @map("updated_at")

  order       Order        @relation(fields: [orderId], references: [id])
  user        User         @relation(fields: [userId], references: [id])
  escrowHolds EscrowHold[]

  @@index([razorpayPaymentId])
  @@map("payments")
}

model EscrowHold {
  id          String       @id @default(uuid()) @db.Uuid
  paymentId   String       @map("payment_id") @db.Uuid
  orderItemId String       @map("order_item_id") @db.Uuid
  sellerId    String       @map("seller_id") @db.Uuid
  amount      Decimal      @db.Decimal(10, 2)
  status      EscrowStatus @default(HELD)
  holdUntil   DateTime     @map("hold_until")
  releasedAt  DateTime?    @map("released_at")
  createdAt   DateTime     @default(now()) @map("created_at")

  payment   Payment   @relation(fields: [paymentId], references: [id])
  orderItem OrderItem @relation(fields: [orderItemId], references: [id])

  @@map("escrow_holds")
}

model SellerPayout {
  id               String       @id @default(uuid()) @db.Uuid
  sellerId         String       @map("seller_id") @db.Uuid
  amount           Decimal      @db.Decimal(10, 2)
  commission       Decimal      @db.Decimal(10, 2)
  netAmount        Decimal      @map("net_amount") @db.Decimal(10, 2)
  razorpayPayoutId String?      @map("razorpay_payout_id") @db.VarChar(100)
  status           PayoutStatus @default(PENDING)
  periodStart      Date         @map("period_start")
  periodEnd        Date         @map("period_end")
  initiatedAt      DateTime     @default(now()) @map("initiated_at")
  completedAt      DateTime?    @map("completed_at")

  seller User @relation(fields: [sellerId], references: [id])

  @@index([sellerId, status])
  @@map("seller_payouts")
}

// ==========================================
// CHAT DOMAIN
// ==========================================

model ChatRoom {
  id            String   @id @default(uuid()) @db.Uuid
  petListingId  String?  @map("pet_listing_id") @db.Uuid
  buyerId       String   @map("buyer_id") @db.Uuid
  sellerId      String   @map("seller_id") @db.Uuid
  lastMessageAt DateTime? @map("last_message_at")
  isActive      Boolean  @default(true) @map("is_active")
  createdAt     DateTime @default(now()) @map("created_at")

  petListing PetListing? @relation(fields: [petListingId], references: [id])
  buyer      User        @relation("BuyerChats", fields: [buyerId], references: [id])
  seller     User        @relation("SellerChats", fields: [sellerId], references: [id])
  messages   Message[]

  @@unique([buyerId, sellerId, petListingId])
  @@map("chat_rooms")
}

model Message {
  id         String      @id @default(uuid()) @db.Uuid
  chatRoomId String      @map("chat_room_id") @db.Uuid
  senderId   String      @map("sender_id") @db.Uuid
  content    String?     @db.Text
  messageType MessageType @default(TEXT) @map("message_type")
  mediaUrl   String?     @map("media_url") @db.VarChar(500)
  isRead     Boolean     @default(false) @map("is_read")
  sentAt     DateTime    @default(now()) @map("sent_at")

  chatRoom ChatRoom @relation(fields: [chatRoomId], references: [id], onDelete: Cascade)
  sender   User     @relation(fields: [senderId], references: [id])

  @@index([chatRoomId, sentAt])
  @@map("messages")
}

// ==========================================
// SERVICE DOMAIN
// ==========================================

model ServiceProvider {
  id                 String             @id @default(uuid()) @db.Uuid
  userId             String             @unique @map("user_id") @db.Uuid
  serviceType        ServiceType        @map("service_type")
  businessName       String?            @map("business_name") @db.VarChar(200)
  licenseNumber      String?            @map("license_number") @db.VarChar(50)
  licenseUrl         String?            @map("license_url") @db.VarChar(500)
  description        String?            @db.Text
  city               String             @db.VarChar(100)
  state              String             @db.VarChar(100)
  latitude           Float?
  longitude          Float?
  servicesOffered    Json?              @map("services_offered") @db.JsonB
  availability       Json?              @db.JsonB
  avgRating          Float              @default(0) @map("avg_rating")
  reviewCount        Int                @default(0) @map("review_count")
  verificationStatus VerificationStatus @default(UNVERIFIED) @map("verification_status")
  isActive           Boolean            @default(true) @map("is_active")
  createdAt          DateTime           @default(now()) @map("created_at")

  user     User      @relation(fields: [userId], references: [id])
  bookings Booking[]

  @@index([serviceType, city, isActive])
  @@map("service_providers")
}

model Booking {
  id          String        @id @default(uuid()) @db.Uuid
  customerId  String        @map("customer_id") @db.Uuid
  providerId  String        @map("provider_id") @db.Uuid
  serviceName String        @map("service_name") @db.VarChar(100)
  bookingDate Date          @map("booking_date")
  startTime   String        @map("start_time") @db.VarChar(5)
  endTime     String        @map("end_time") @db.VarChar(5)
  price       Decimal       @db.Decimal(10, 2)
  status      BookingStatus @default(PENDING)
  notes       String?       @db.VarChar(500)
  createdAt   DateTime      @default(now()) @map("created_at")
  updatedAt   DateTime      @updatedAt @map("updated_at")

  customer User            @relation("CustomerBookings", fields: [customerId], references: [id])
  provider ServiceProvider @relation(fields: [providerId], references: [id])

  @@index([providerId, bookingDate])
  @@index([customerId])
  @@map("bookings")
}

// ==========================================
// REVIEW DOMAIN
// ==========================================

model Review {
  id                 String           @id @default(uuid()) @db.Uuid
  reviewerId         String           @map("reviewer_id") @db.Uuid
  targetId           String           @map("target_id") @db.Uuid
  targetType         ReviewTargetType @map("target_type")
  rating             Int
  content            String?          @db.Text
  photos             Json?            @db.JsonB
  sellerResponse     String?          @map("seller_response") @db.Text
  isVerifiedPurchase Boolean          @default(false) @map("is_verified_purchase")
  helpfulCount       Int              @default(0) @map("helpful_count")
  createdAt          DateTime         @default(now()) @map("created_at")
  updatedAt          DateTime         @updatedAt @map("updated_at")

  reviewer User @relation("Reviewer", fields: [reviewerId], references: [id])

  @@index([targetId, targetType])
  @@map("reviews")
}

// ==========================================
// NOTIFICATION DOMAIN
// ==========================================

model Notification {
  id        String           @id @default(uuid()) @db.Uuid
  userId    String           @map("user_id") @db.Uuid
  title     String           @db.VarChar(200)
  body      String           @db.Text
  type      NotificationType
  data      Json?            @db.JsonB
  isRead    Boolean          @default(false) @map("is_read")
  createdAt DateTime         @default(now()) @map("created_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId, isRead, createdAt])
  @@map("notifications")
}

// ==========================================
// FRANCHISE DOMAIN
// ==========================================

model Franchise {
  id              String          @id @default(uuid()) @db.Uuid
  ownerId         String          @map("owner_id") @db.Uuid
  name            String          @db.VarChar(200)
  city            String          @db.VarChar(100)
  state           String          @db.VarChar(100)
  address         String          @db.Text
  status          FranchiseStatus @default(APPLIED)
  revenueSharePct Decimal         @map("revenue_share_pct") @db.Decimal(5, 2)
  onboardedAt     DateTime?       @map("onboarded_at")
  createdAt       DateTime        @default(now()) @map("created_at")

  owner User @relation(fields: [ownerId], references: [id])

  @@map("franchises")
}

// ==========================================
// SYSTEM TABLES
// ==========================================

model Coupon {
  id             String   @id @default(uuid()) @db.Uuid
  code           String   @unique @db.VarChar(20)
  description    String?  @db.VarChar(200)
  discountType   String   @map("discount_type") @db.VarChar(20) // PERCENTAGE or FLAT
  discountValue  Decimal  @map("discount_value") @db.Decimal(10, 2)
  minOrderValue  Decimal? @map("min_order_value") @db.Decimal(10, 2)
  maxDiscount    Decimal? @map("max_discount") @db.Decimal(10, 2)
  usageLimit     Int?     @map("usage_limit")
  usedCount      Int      @default(0) @map("used_count")
  validFrom      DateTime @map("valid_from")
  validUntil     DateTime @map("valid_until")
  isActive       Boolean  @default(true) @map("is_active")

  @@map("coupons")
}

model AuditLog {
  id         String   @id @default(uuid()) @db.Uuid
  userId     String   @map("user_id") @db.Uuid
  action     String   @db.VarChar(100)
  entityType String   @map("entity_type") @db.VarChar(50)
  entityId   String   @map("entity_id") @db.Uuid
  changes    Json?    @db.JsonB
  ipAddress  String?  @map("ip_address") @db.VarChar(45)
  createdAt  DateTime @default(now()) @map("created_at")

  @@index([userId])
  @@index([entityType, entityId])
  @@map("audit_logs")
}

// ==========================================
// COMMUNITY & FORUMS
// ==========================================

enum ForumCategory {
  BREED_DISCUSSION
  HEALTH_NUTRITION
  TRAINING_TIPS
  LOST_AND_FOUND
  BUY_SELL_ADVICE
  GENERAL
}

enum PostStatus {
  ACTIVE
  HIDDEN
  LOCKED
  DELETED
}

model ForumPost {
  id          String        @id @default(uuid()) @db.Uuid
  authorId    String        @map("author_id") @db.Uuid
  author      User          @relation(fields: [authorId], references: [id])
  category    ForumCategory
  title       String        @db.VarChar(200)
  body        String        @db.Text
  images      String[]      @db.Text
  species     String?       @db.VarChar(50)
  breed       String?       @db.VarChar(100)
  tags        String[]      @db.VarChar(50)
  status      PostStatus    @default(ACTIVE)
  isPinned    Boolean       @default(false) @map("is_pinned")
  viewCount   Int           @default(0) @map("view_count")
  replyCount  Int           @default(0) @map("reply_count")
  upvotes     Int           @default(0)
  downvotes   Int           @default(0)
  replies     ForumReply[]
  votes       ForumVote[]
  createdAt   DateTime      @default(now()) @map("created_at")
  updatedAt   DateTime      @updatedAt @map("updated_at")

  @@index([authorId])
  @@index([category])
  @@index([species])
  @@index([createdAt])
  @@map("forum_posts")
}

model ForumReply {
  id        String     @id @default(uuid()) @db.Uuid
  postId    String     @map("post_id") @db.Uuid
  post      ForumPost  @relation(fields: [postId], references: [id])
  authorId  String     @map("author_id") @db.Uuid
  author    User       @relation(fields: [authorId], references: [id])
  parentId  String?    @map("parent_id") @db.Uuid
  parent    ForumReply? @relation("ReplyToReply", fields: [parentId], references: [id])
  children  ForumReply[] @relation("ReplyToReply")
  body      String     @db.Text
  images    String[]   @db.Text
  upvotes   Int        @default(0)
  downvotes Int        @default(0)
  status    PostStatus @default(ACTIVE)
  votes     ForumVote[]
  createdAt DateTime   @default(now()) @map("created_at")
  updatedAt DateTime   @updatedAt @map("updated_at")

  @@index([postId])
  @@index([authorId])
  @@index([parentId])
  @@map("forum_replies")
}

model ForumVote {
  id       String      @id @default(uuid()) @db.Uuid
  userId   String      @map("user_id") @db.Uuid
  user     User        @relation(fields: [userId], references: [id])
  postId   String?     @map("post_id") @db.Uuid
  post     ForumPost?  @relation(fields: [postId], references: [id])
  replyId  String?     @map("reply_id") @db.Uuid
  reply    ForumReply? @relation(fields: [replyId], references: [id])
  value    Int         // +1 or -1

  @@unique([userId, postId])
  @@unique([userId, replyId])
  @@map("forum_votes")
}

model ForumFollowing {
  id        String   @id @default(uuid()) @db.Uuid
  userId    String   @map("user_id") @db.Uuid
  user      User     @relation(fields: [userId], references: [id])
  postId    String   @map("post_id") @db.Uuid
  createdAt DateTime @default(now()) @map("created_at")

  @@unique([userId, postId])
  @@map("forum_followings")
}

model LostFoundPost {
  id          String   @id @default(uuid()) @db.Uuid
  authorId    String   @map("author_id") @db.Uuid
  author      User     @relation(fields: [authorId], references: [id])
  type        String   @db.VarChar(10) // LOST or FOUND
  petName     String?  @map("pet_name") @db.VarChar(100)
  species     String   @db.VarChar(50)
  breed       String?  @db.VarChar(100)
  color       String?  @db.VarChar(100)
  description String   @db.Text
  images      String[] @db.Text
  lastSeenAt  String?  @map("last_seen_at") @db.VarChar(200)
  latitude    Float?
  longitude   Float?
  city        String   @db.VarChar(100)
  contactPhone String? @map("contact_phone") @db.VarChar(15)
  isResolved  Boolean  @default(false) @map("is_resolved")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  @@index([city])
  @@index([species])
  @@index([isResolved])
  @@map("lost_found_posts")
}

// ==========================================
// EDUCATIONAL CONTENT & TRAINING
// ==========================================

enum ContentType {
  ARTICLE
  VIDEO
  COURSE
  GUIDE
}

enum ContentStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

model EducationalContent {
  id            String        @id @default(uuid()) @db.Uuid
  authorId      String        @map("author_id") @db.Uuid
  author        User          @relation(fields: [authorId], references: [id])
  type          ContentType
  title         String        @db.VarChar(200)
  slug          String        @unique @db.VarChar(250)
  description   String        @db.Text
  body          String?       @db.Text // For articles/guides
  videoUrl      String?       @map("video_url") @db.VarChar(500) // For videos
  thumbnailUrl  String?       @map("thumbnail_url") @db.VarChar(500)
  duration      Int?          // Video duration in seconds
  category      String        @db.VarChar(100) // Grooming, Training, Nutrition, Health, Breed-specific
  species       String?       @db.VarChar(50)
  breed         String?       @db.VarChar(100)
  difficulty    String?       @db.VarChar(20) // beginner, intermediate, advanced
  tags          String[]      @db.VarChar(50)
  status        ContentStatus @default(DRAFT)
  viewCount     Int           @default(0) @map("view_count")
  likeCount     Int           @default(0) @map("like_count")
  isFeatured    Boolean       @default(false) @map("is_featured")
  isPremium     Boolean       @default(false) @map("is_premium")
  courseId      String?       @map("course_id") @db.Uuid
  course        Course?       @relation(fields: [courseId], references: [id])
  orderInCourse Int?          @map("order_in_course")
  createdAt     DateTime      @default(now()) @map("created_at")
  updatedAt     DateTime      @updatedAt @map("updated_at")

  @@index([type, category])
  @@index([species])
  @@index([status])
  @@index([courseId])
  @@map("educational_content")
}

model Course {
  id          String              @id @default(uuid()) @db.Uuid
  authorId    String              @map("author_id") @db.Uuid
  author      User                @relation(fields: [authorId], references: [id])
  title       String              @db.VarChar(200)
  slug        String              @unique @db.VarChar(250)
  description String              @db.Text
  thumbnail   String?             @db.VarChar(500)
  category    String              @db.VarChar(100)
  species     String?             @db.VarChar(50)
  difficulty  String              @db.VarChar(20)
  totalVideos Int                 @default(0) @map("total_videos")
  totalDuration Int               @default(0) @map("total_duration") // seconds
  isPremium   Boolean             @default(false) @map("is_premium")
  price       Decimal?            @db.Decimal(10, 2)
  status      ContentStatus       @default(DRAFT)
  contents    EducationalContent[]
  enrollments CourseEnrollment[]
  createdAt   DateTime            @default(now()) @map("created_at")
  updatedAt   DateTime            @updatedAt @map("updated_at")

  @@index([category])
  @@index([species])
  @@map("courses")
}

model CourseEnrollment {
  id              String   @id @default(uuid()) @db.Uuid
  userId          String   @map("user_id") @db.Uuid
  user            User     @relation(fields: [userId], references: [id])
  courseId         String   @map("course_id") @db.Uuid
  course          Course   @relation(fields: [courseId], references: [id])
  progress        Int      @default(0) // percentage 0-100
  lastContentId   String?  @map("last_content_id") @db.Uuid
  completedAt     DateTime? @map("completed_at")
  certificateUrl  String?  @map("certificate_url") @db.VarChar(500)
  createdAt       DateTime @default(now()) @map("created_at")
  updatedAt       DateTime @updatedAt @map("updated_at")

  @@unique([userId, courseId])
  @@map("course_enrollments")
}

model VetConsultation {
  id            String   @id @default(uuid()) @db.Uuid
  userId        String   @map("user_id") @db.Uuid
  user          User     @relation(fields: [userId], references: [id])
  vetId         String   @map("vet_id") @db.Uuid
  vet           User     @relation("VetConsultations", fields: [vetId], references: [id])
  type          String   @db.VarChar(20) // video, chat, qa
  petName       String?  @map("pet_name") @db.VarChar(100)
  petSpecies    String?  @map("pet_species") @db.VarChar(50)
  petBreed      String?  @map("pet_breed") @db.VarChar(100)
  symptoms      String?  @db.Text
  diagnosis     String?  @db.Text
  prescription  Json?    @db.JsonB
  attachments   String[] @db.Text
  duration      Int?     // minutes
  status        String   @db.VarChar(20) // scheduled, in_progress, completed, cancelled
  scheduledAt   DateTime? @map("scheduled_at")
  startedAt     DateTime? @map("started_at")
  endedAt       DateTime? @map("ended_at")
  amount        Decimal  @db.Decimal(10, 2)
  paymentId     String?  @map("payment_id") @db.Uuid
  rating        Int?
  createdAt     DateTime @default(now()) @map("created_at")
  updatedAt     DateTime @updatedAt @map("updated_at")

  @@index([userId])
  @@index([vetId])
  @@index([status])
  @@map("vet_consultations")
}

// ==========================================
// PET INSURANCE
// ==========================================

enum InsurancePolicyStatus {
  ACTIVE
  EXPIRED
  CANCELLED
  CLAIMED
}

enum ClaimStatus {
  SUBMITTED
  UNDER_REVIEW
  APPROVED
  REJECTED
  PAID
}

model InsurancePartner {
  id          String   @id @default(uuid()) @db.Uuid
  name        String   @db.VarChar(200)
  slug        String   @unique @db.VarChar(250)
  logo        String?  @db.VarChar(500)
  description String?  @db.Text
  website     String?  @db.VarChar(500)
  contactEmail String? @map("contact_email") @db.VarChar(200)
  contactPhone String? @map("contact_phone") @db.VarChar(15)
  isActive    Boolean  @default(true) @map("is_active")
  plans       InsurancePlan[]
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  @@map("insurance_partners")
}

model InsurancePlan {
  id              String           @id @default(uuid()) @db.Uuid
  partnerId       String           @map("partner_id") @db.Uuid
  partner         InsurancePartner @relation(fields: [partnerId], references: [id])
  name            String           @db.VarChar(200)
  description     String           @db.Text
  coverageType    String           @map("coverage_type") @db.VarChar(50) // basic, comprehensive, accident_only
  speciesCovered  String[]         @map("species_covered") @db.VarChar(50)
  coverageAmount  Decimal          @map("coverage_amount") @db.Decimal(10, 2)
  premiumMonthly  Decimal          @map("premium_monthly") @db.Decimal(10, 2)
  premiumYearly   Decimal          @map("premium_yearly") @db.Decimal(10, 2)
  deductible      Decimal?         @db.Decimal(10, 2)
  waitingPeriod   Int?             @map("waiting_period") // days
  maxAge          Int?             @map("max_age") // years
  coverageDetails Json             @map("coverage_details") @db.JsonB // what's covered
  exclusions      Json?            @db.JsonB // what's not covered
  claimProcess    String?          @map("claim_process") @db.Text
  isActive        Boolean          @default(true) @map("is_active")
  policies        InsurancePolicy[]
  createdAt       DateTime         @default(now()) @map("created_at")
  updatedAt       DateTime         @updatedAt @map("updated_at")

  @@index([partnerId])
  @@index([coverageType])
  @@map("insurance_plans")
}

model InsurancePolicy {
  id              String                @id @default(uuid()) @db.Uuid
  userId          String                @map("user_id") @db.Uuid
  user            User                  @relation(fields: [userId], references: [id])
  planId          String                @map("plan_id") @db.Uuid
  plan            InsurancePlan         @relation(fields: [planId], references: [id])
  policyNumber    String                @unique @map("policy_number") @db.VarChar(50)
  petName         String                @map("pet_name") @db.VarChar(100)
  petSpecies      String                @map("pet_species") @db.VarChar(50)
  petBreed        String                @map("pet_breed") @db.VarChar(100)
  petAge          Int                   @map("pet_age") // months
  petGender       Gender                @map("pet_gender")
  startDate       DateTime              @map("start_date")
  endDate         DateTime              @map("end_date")
  premiumPaid     Decimal               @map("premium_paid") @db.Decimal(10, 2)
  paymentFrequency String              @map("payment_frequency") @db.VarChar(20) // monthly, yearly
  status          InsurancePolicyStatus @default(ACTIVE)
  documentUrl     String?               @map("document_url") @db.VarChar(500)
  claims          InsuranceClaim[]
  createdAt       DateTime              @default(now()) @map("created_at")
  updatedAt       DateTime              @updatedAt @map("updated_at")

  @@index([userId])
  @@index([planId])
  @@index([status])
  @@map("insurance_policies")
}

model InsuranceClaim {
  id            String      @id @default(uuid()) @db.Uuid
  policyId      String      @map("policy_id") @db.Uuid
  policy        InsurancePolicy @relation(fields: [policyId], references: [id])
  claimNumber   String      @unique @map("claim_number") @db.VarChar(50)
  incidentDate  DateTime    @map("incident_date")
  description   String      @db.Text
  claimAmount   Decimal     @map("claim_amount") @db.Decimal(10, 2)
  approvedAmount Decimal?   @map("approved_amount") @db.Decimal(10, 2)
  documents     String[]    @db.Text // bills, reports, photos
  status        ClaimStatus @default(SUBMITTED)
  rejectionReason String?   @map("rejection_reason") @db.Text
  processedAt   DateTime?   @map("processed_at")
  paidAt        DateTime?   @map("paid_at")
  createdAt     DateTime    @default(now()) @map("created_at")
  updatedAt     DateTime    @updatedAt @map("updated_at")

  @@index([policyId])
  @@index([status])
  @@map("insurance_claims")
}
```

---

## Table Count Summary

| Domain | Tables |
|--------|--------|
| User | 5 (users, profiles, roles, devices, kyc) |
| Pet Marketplace | 4 (species, breeds, listings, media, vaccinations) |
| Breeder | 3 (profiles, parents, litters) |
| Product Store | 4 (categories, products, variants, images) |
| Order | 4 (carts, cart_items, orders, order_items, addresses) |
| Payment | 3 (payments, escrow_holds, seller_payouts) |
| Chat | 2 (chat_rooms, messages) |
| Service | 2 (service_providers, bookings) |
| Review | 1 (reviews — polymorphic) |
| Notification | 1 (notifications) |
| Franchise | 1 (franchises) |
| System | 2 (coupons, audit_logs) |
| **Total** | **~32 tables** |
