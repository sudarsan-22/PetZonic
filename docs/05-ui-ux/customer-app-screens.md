# PetZonic — Customer App Screens (Flutter)

> **Version**: 1.0.0  
> **Platform**: iOS & Android (Flutter)

---

## 1. Launch & Authentication

### 1.1 Splash Screen
- PetZonic logo with paw animation
- Auto-checks auth token validity
- Routes to Home (logged in) or Onboarding (new install)

### 1.2 Onboarding (3 slides)
- **Slide 1**: "Find Your Perfect Pet" — marketplace illustration
- **Slide 2**: "Everything They Need" — product store illustration
- **Slide 3**: "Expert Care Nearby" — services illustration
- Skip button (top-right), Get Started button (final slide)

### 1.3 Login Screen
- Phone number input (with country code +91 default)
- "Send OTP" button
- Divider: "or continue with"
- Google Sign-In button
- Apple Sign-In button (iOS only)
- Email login option (expandable)
- Terms & Privacy links at bottom

### 1.4 OTP Verification
- 6-digit OTP input boxes (auto-advance)
- Auto-read SMS OTP (Android)
- 30-second countdown timer for resend
- "Resend OTP" button (enabled after timer)
- "Change number" link

### 1.5 Profile Setup (New Users)
- Profile photo (camera/gallery)
- Full name input
- City selector (autocomplete)
- "What pets interest you?" — multi-select chips (Dogs, Cats, Birds, Fish, Others)
- "I also want to sell pets" toggle → shows seller type selector
- Skip / Complete button

---

## 2. Home & Navigation

### 2.1 Home Screen
- **Top bar**: City selector, search icon, notification bell (badge)
- **Search bar**: "Search pets, products, services..."
- **Banner carousel**: Promotional banners (auto-scroll)
- **Quick categories**: Horizontal scroll — Dogs, Cats, Birds, Fish, Products, Services
- **Featured Pets**: Horizontal card scroll (2.5 visible)
- **Deals of the Day**: Timer + product cards
- **Popular Products**: Grid (2 columns)
- **Services Near You**: Horizontal provider cards
- **Recently Viewed**: If returning user

### 2.2 Bottom Navigation Bar
- 🏠 Home
- 🐾 Pets (Marketplace)
- 🛒 Cart (badge count)
- 💬 Chat (unread badge)
- 👤 Profile

---

## 3. Pet Marketplace

### 3.1 Pet Listings Screen
- **Top**: Tab chips — All, Dogs, Cats, Birds, Fish, Exotic
- **Filter button** (bottom sheet on tap)
- **Sort dropdown**: Newest, Price ↑, Price ↓, Distance
- **Grid view**: 2-column cards
  - Card: Image, species badge, breed, price, location, seller badge
- **Empty state**: "No pets found. Try different filters."
- Pull-to-refresh, infinite scroll

### 3.2 Pet Filters (Bottom Sheet)
- **Species**: Multi-select chips
- **Breed**: Searchable dropdown (auto-populated based on species)
- **Price range**: Dual slider (₹500 — ₹5,00,000)
- **Gender**: Male / Female / Any
- **Age**: Puppy/Kitten, Young, Adult
- **Location**: City selector + radius slider (1-50 km)
- **Seller type**: Individual, Breeder, Shop
- **Verified sellers only**: Toggle
- **Apply / Reset buttons**

### 3.3 Pet Detail Screen
- **Image gallery**: Full-width carousel + thumbnail strip + video indicator
- **Quick actions**: ❤️ Wishlist, Share, Report
- **Pet info card**: Breed, age, gender, color, weight, vaccinated badge
- **Price section**: Price + "Negotiate" tag if applicable
- **About section**: Description text (expandable)
- **Health & Docs**: Vaccination list with dates, health certificate indicator
- **Parent info**: If breeder listing (father/mother photos + details)
- **Seller card**: Avatar, name, rating, verified badge, "View Profile" link
- **Location**: City + approximate map (no exact address)
- **Similar pets**: Horizontal scroll
- **Action bar (sticky bottom)**: "Chat with Seller" + "Buy Now"

### 3.4 Seller Public Profile
- Cover photo + avatar
- Seller name, rating (stars + count), verified badge
- Member since, response rate, response time
- About section
- Active listings grid
- Reviews summary + recent reviews
- Report seller button

---

## 4. Product Store

### 4.1 Products Home
- Category icons grid (Food, Toys, Accessories, Health, Grooming)
- Featured brands horizontal scroll
- Flash deals section with timer
- "New Arrivals" section
- "Best Sellers" section

### 4.2 Product Listing
- Category breadcrumb
- Filter chips (Quick: In Stock, On Sale, Top Rated)
- Sort dropdown
- Grid (2 columns):
  - Card: Image, brand, name, MRP (strikethrough), selling price, discount %, rating stars, "Add to Cart" button
- Infinite scroll

### 4.3 Product Detail
- Image carousel (pinch to zoom)
- Brand name (tappable → brand page)
- Product name
- Rating: stars + count (tappable → reviews)
- Price: MRP strikethrough, selling price, discount %, "Inclusive of taxes"
- Variant selector: Size/flavor chips
- Stock: "In Stock" (green) or "Only 3 left!" (orange)
- "Add to Cart" / "Go to Cart" button
- Delivery estimate: "Delivers by May 31" (based on pincode)
- Pincode checker input
- Description (collapsible)
- Specifications table
- Reviews section (top 3 + "View All")
- Related products horizontal scroll

### 4.4 Product Reviews
- Rating summary bar
- Filter chips: 5⭐, 4⭐, 3⭐, 2⭐, 1⭐, With Photos
- Review cards: avatar, name, rating, date, title, body, photos, helpful count
- Sort: Most Helpful, Newest, Highest, Lowest

---

## 5. Cart & Checkout

### 5.1 Cart Screen
- Cart items list:
  - Product image, name, variant, price
  - Quantity stepper (- / count / +)
  - Remove button
  - "Move to Wishlist" option
- Coupon section: "Apply Coupon" expandable
- Price breakdown: Subtotal, Discount, Delivery, Total
- "Proceed to Checkout" button (sticky)
- Empty cart state: illustration + "Start Shopping" CTA

### 5.2 Address Selection
- Saved addresses list (radio select)
- Each: Name, full address, phone, type badge (Home/Work/Other)
- "Add New Address" card
- "Deliver Here" button

### 5.3 Add/Edit Address
- Full name
- Phone number
- Pincode (auto-fills city/state)
- Address line 1, line 2
- City (auto-filled)
- State (auto-filled)
- Type: Home / Work / Other
- "Set as default" toggle
- Map pin (optional - tap to set location)
- Save button

### 5.4 Delivery Slot Selection
- Delivery estimate per item
- Slot picker: dates (horizontal scroll) + time windows (Morning, Afternoon, Evening)
- Express delivery option (if available, extra charge)

### 5.5 Payment & Order Summary
- Order items summary (collapsed)
- Delivery address (change link)
- Price breakdown
- Payment options:
  - UPI (Google Pay, PhonePe, others)
  - Cards (saved + add new)
  - Net Banking
  - Wallets
  - Cash on Delivery (if eligible)
- "Place Order" button
- Razorpay payment sheet opens on tap

### 5.6 Order Success
- ✅ Checkmark animation
- "Order Placed Successfully!"
- Order number
- Estimated delivery
- "Track Order" button
- "Continue Shopping" button

---

## 6. Orders

### 6.1 My Orders
- Tabs: Active, Delivered, Cancelled
- Order cards:
  - Order number, date, item count, total
  - Status chip (Processing, Shipped, Delivered, Cancelled)
  - First item thumbnail
  - "Track" / "Reorder" / "Review" action
- Empty state per tab

### 6.2 Order Detail
- Order status timeline (stepper): Placed → Confirmed → Shipped → Out for Delivery → Delivered
- Items list with thumbnails
- Delivery address
- Payment info (method, transaction ID)
- Price breakdown
- Actions: "Track Shipment", "Cancel Order", "Need Help"

### 6.3 Pet Order Detail (Escrow)
- Status: Payment Held in Escrow → Coordinating → Pet Received → Payment Released
- Chat button (with seller)
- "Confirm Pet Received" button
- "Report Issue" button
- 48-hour confirmation window indicator

---

## 7. Services

### 7.1 Services Home
- Service categories: Vet, Grooming, Training, Sitting, Walking, Boarding
- "Near You" section (location-based cards)
- "Top Rated" section
- Search bar for providers

### 7.2 Provider List
- Provider cards:
  - Photo, name, type badge, rating
  - Distance, price starting from
  - "Book Now" button
- Map toggle (list view ↔ map view with pins)
- Filter: distance, rating, price, species served, availability

### 7.3 Provider Detail
- Cover photo gallery
- Name, type, verified badge
- Rating (stars + count)
- Address + map preview + "Get Directions"
- Operating hours (today highlighted)
- Services offered list (name, duration, price)
- "Book Appointment" button
- About / Bio section
- Credentials & certificates
- Reviews section
- Gallery

### 7.4 Booking Flow
- Service selected (changeable)
- Date picker (calendar, disabled dates grayed)
- Time slots grid (available in blue, taken grayed)
- Pet info form (select from my pets or add new)
- Notes/symptoms text area
- Booking summary
- "Confirm & Pay" button

### 7.5 My Bookings
- Tabs: Upcoming, Past, Cancelled
- Booking cards: provider, service, date/time, status
- Actions: Cancel, Reschedule, Add to Calendar

---

## 8. Chat

### 8.1 Chat List
- Conversation list sorted by recent
- Each: Avatar, name (+ seller badge), last message preview, time, unread count
- Search conversations
- Empty state: "Start chatting with pet sellers!"

### 8.2 Chat Room
- Header: Seller name, online status, listing context
- Message bubbles (sent/received)
- Types: Text, image, location, offer
- Input bar: Text input, attach (camera, gallery, location), send button
- Listing card at top (tappable → opens listing)
- "Make Offer" quick action

---

## 9. Engagement

### 9.1 Notifications
- Grouped by today, yesterday, earlier
- Cards: icon, title, body, time, read/unread indicator
- Tap navigates to relevant screen
- "Mark all read" action

### 9.2 Wishlist
- Tabs: Pets, Products
- Same cards as listing/product pages with remove button
- "Price dropped!" badge on discounted items
- Empty state per tab

### 9.3 Search
- Recent searches (clearable)
- Trending searches
- Results tabbed: Pets, Products, Services, Sellers
- Auto-suggest as you type (debounced)

---

## 10. Account

### 10.1 Profile Screen
- Avatar + name + member since
- Section links:
  - My Orders
  - My Wishlist
  - My Addresses
  - Payment Methods
  - Notification Settings
  - Help & Support
  - About PetZonic
  - Rate the App
  - Logout
- "Become a Seller" banner (if not seller)

### 10.2 Edit Profile
- Avatar (camera/gallery)
- Name
- Email (add/verify)
- Phone (change with OTP)
- City
- Bio

### 10.3 Help & Support
- FAQ accordion sections
- Search FAQs
- "Contact Us" → opens email/chat
- "Report a Problem" form
