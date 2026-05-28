# PetZonic — Website Pages (Next.js)

> **Version**: 1.0.0  
> **Framework**: Next.js 15 (App Router)

---

## 1. Public Pages (No Auth Required)

### 1.1 Homepage (`/`)
- **Hero section**: Full-width banner with CTA "Find Your Perfect Pet"
- **Search bar**: Prominent search with category tabs (Pets / Products / Services)
- **Category tiles**: Visual grid — Dogs, Cats, Birds, Fish, Exotic, Products, Services
- **Featured pets**: Card carousel (6 featured listings)
- **How it works**: 3-step infographic (Browse → Connect → Adopt/Buy)
- **Product deals**: Grid of discounted products with timer
- **Services near you**: Provider cards (requires location)
- **Testimonials**: Customer review carousel
- **Download app CTA**: App store buttons + QR code
- **Footer**: Links, social media, newsletter signup

### 1.2 Pet Listings (`/pets`)
- **SEO title**: "Buy Pets Online in India | Dogs, Cats & More — PetZonic"
- **Breadcrumb**: Home > Pets
- **Sidebar filters** (desktop): Species, breed, price range, gender, age, city, verified
- **Top bar**: Sort (Newest, Price ↑↓, Distance), view toggle (grid/list)
- **Listing cards**: Photo, breed, age, gender, price, city, seller badge
- **Pagination**: Numbered pages + prev/next (SEO-friendly)
- **Mobile**: Filters in slide-out drawer

### 1.3 Pet Detail (`/pets/[slug]`)
- **SEO**: Dynamic meta title "Golden Retriever Puppy for Sale in Bangalore — PetZonic"
- **Image gallery**: Thumbnails + lightbox + video tab
- **Pet info**: Structured data card (breed, age, gender, color, weight, vaccinations)
- **Price**: Prominently displayed with "Chat to Negotiate" option
- **Description**: Seller's detailed description
- **Health section**: Vaccination table, health certificates
- **Seller card**: Avatar, name, rating, "View Profile" link, "Contact Seller" button
- **Location**: City + area (map preview, no exact address)
- **Similar listings**: Grid below
- **CTA bar (sticky)**: "Chat with Seller" / "Buy Now"
- **Schema.org**: Product structured data for Google

### 1.4 Products (`/products`)
- **SEO**: Category-based titles
- **Category navigation**: Sidebar tree (desktop), horizontal scroll (mobile)
- **Product grid**: Cards with image, brand, name, price (MRP crossed), rating, Add to Cart
- **Filters**: Brand, price range, rating, species, in stock
- **Sort**: Popular, Newest, Price, Rating

### 1.5 Product Detail (`/products/[slug]`)
- **Breadcrumb**: Home > Products > Category > Product
- **Image carousel**: Multiple images, zoom on hover
- **Product info**: Brand, name, rating (stars + count)
- **Price block**: MRP strikethrough, selling price, discount %, EMI option
- **Variant selector**: Radio/chips for size/flavor/color
- **Quantity + Add to Cart** button
- **Delivery check**: Pincode input → "Delivers by [date]"
- **Description tabs**: Description, Specifications, Reviews
- **Reviews section**: Summary + individual reviews
- **Frequently bought together**: Cross-sell section
- **Schema.org**: Product + AggregateRating structured data

### 1.6 Services (`/services`)
- **Service categories**: Visual cards (Vet, Grooming, Training, Sitting, Walking)
- **"Near You" section**: Requires location permission
- **Provider cards**: Photo, name, rating, distance, starting price, "Book" button
- **Search by city**: For non-location users

### 1.7 Provider Detail (`/services/[id]`)
- **Header**: Cover photo, profile image, name, type badge, rating
- **About**: Bio, experience, credentials
- **Services offered**: Table (name, duration, price, "Book" button)
- **Availability calendar**: Weekly view
- **Gallery**: Facility/work photos
- **Reviews**: Full review section
- **Location**: Map embed + "Get Directions" link
- **Book Now CTA**: Opens booking flow

### 1.8 Breed Guides (`/breeds/[breed]`)
- **SEO content pages**: "Golden Retriever — Complete Guide | PetZonic"
- **Breed overview**: Photo, origin, size, temperament
- **Care guide**: Grooming, exercise, diet, health concerns
- **Available listings**: Pets of this breed currently for sale
- **Recommended products**: Breed-specific product suggestions
- **FAQ**: Common questions about the breed
- Schema.org: Article structured data

### 1.9 About Us (`/about`)
- Company story, mission, team
- Stats: pets rehomed, sellers, cities
- How it works section

### 1.10 Contact (`/contact`)
- Contact form (name, email, subject, message)
- Email address, phone number
- Office address + map
- Social media links

### 1.11 FAQ (`/faq`)
- Categorized accordion: Buying, Selling, Payments, Delivery, Account
- Search FAQs
- "Still need help?" → Contact form

### 1.12 Legal Pages
- **Terms of Service** (`/terms`): Full legal terms
- **Privacy Policy** (`/privacy`): GDPR-compatible policy
- **Refund Policy** (`/refund-policy`): Escrow + product return policy

### 1.13 Seller Landing (`/sell`)
- "Become a Seller on PetZonic"
- Benefits: reach, escrow safety, analytics
- How to get started (3 steps)
- Success stories / testimonials
- "Start Selling" CTA → auth → seller onboarding
- FAQ for sellers

---

## 2. Authenticated Pages (Login Required)

### 2.1 My Account (`/account`)
- Account overview card (name, email, member since)
- Quick links: Orders, Wishlist, Addresses, Settings
- Recent orders summary

### 2.2 My Orders (`/account/orders`)
- Tab: All, Processing, Shipped, Delivered, Cancelled
- Order cards: Number, date, items, total, status badge
- Click → Order detail

### 2.3 Order Detail (`/account/orders/[id]`)
- Status timeline stepper
- Items ordered (images, names, quantities, prices)
- Delivery info (address, tracking number)
- Payment summary
- Actions: Track, Cancel, Return, Help

### 2.4 My Wishlist (`/account/wishlist`)
- Tabs: Pets, Products
- Grid view of saved items
- "Add to Cart" for products
- "View Listing" for pets
- Remove button

### 2.5 My Addresses (`/account/addresses`)
- Address cards (edit, delete, set default)
- "Add New Address" form

### 2.6 Account Settings (`/account/settings`)
- Edit profile (name, phone, email)
- Change password (if email auth)
- Notification preferences
- Delete account

### 2.7 Cart (`/cart`)
- Item list: image, name, variant, price, quantity controls, remove
- Coupon code input
- Price summary (subtotal, discount, delivery, total)
- "Proceed to Checkout" button
- "Continue Shopping" link
- Recommended products

### 2.8 Checkout (`/checkout`)
- Step 1: Address (select or add)
- Step 2: Delivery (slot selection)
- Step 3: Payment (Razorpay integration)
- Order summary sidebar (sticky on desktop)

---

## 3. Seller Portal (Web — `/seller`)

### 3.1 Seller Dashboard (`/seller`)
- Revenue summary, active listings, pending orders
- Quick actions: Create listing, view orders

### 3.2 Seller Listings (`/seller/listings`)
- Table: Image, title, status, views, inquiries, actions
- Create/edit listing form

### 3.3 Seller Orders (`/seller/orders`)
- Table: Order#, buyer, pet, status, amount, actions
- Accept/decline for new orders

### 3.4 Seller Payouts (`/seller/payouts`)
- Balance, payout history, bank account settings

---

## 4. SEO Strategy

| Page | Target Keywords | Type |
|------|----------------|------|
| Homepage | buy pets online India, pet store | Brand + Category |
| `/pets` | dogs for sale, cats for sale, buy puppy | Category |
| `/pets/[slug]` | [breed] for sale in [city] | Long-tail |
| `/products` | pet food online, dog accessories | Category |
| `/products/[slug]` | [product name] price, buy [brand] | Product |
| `/breeds/[breed]` | [breed] price, [breed] care guide | Informational |
| `/services` | vet near me, pet grooming [city] | Local |
| `/sell` | sell pets online, become pet seller | Transactional |

### SEO Implementation
- Server-side rendering (SSR) for all public pages
- Dynamic sitemap.xml generation
- Schema.org structured data (Product, LocalBusiness, Article, BreadcrumbList)
- Open Graph + Twitter Card meta for social sharing
- Canonical URLs to prevent duplicates
- Image alt tags with descriptive text
- Internal linking strategy between breed guides ↔ listings ↔ products
