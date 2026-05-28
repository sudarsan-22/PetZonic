# PetZonic — Seller App Screens (Flutter)

> **Version**: 1.0.0  
> **Platform**: iOS & Android (Flutter)

---

## 1. Authentication & Onboarding

### 1.1 Seller Login
- Same auth flow as customer app (OTP-based)
- Auto-detects seller role after login
- If new user → routes to seller registration

### 1.2 Seller Registration
- Step 1: Personal info (name, phone verified, email)
- Step 2: Seller type (Individual Seller / Breeder / Pet Shop)
- Step 3: Business details (shop name, address, description)
- Step 4: KYC document upload prompt (can skip → do later)
- "Start Selling" CTA

---

## 2. Dashboard

### 2.1 Seller Dashboard
- **Top bar**: Profile avatar, notification bell (badge)
- **Welcome banner**: "Good morning, [Name]! Here's your summary"
- **Quick stats row**:
  - Active Listings count
  - Pending Orders count
  - This Month Revenue (₹)
  - Average Rating
- **Action cards**:
  - "New Order!" alert (if any pending)
  - "Create New Listing" shortcut
- **Recent activity feed**: Last 5 events (order received, review, listing approved)
- **Chart**: Weekly revenue trend (7-day bar chart)
- **Performance card**: Response rate %, response time avg

### 2.2 Bottom Navigation
- 📊 Dashboard
- 📋 Listings
- 📦 Orders
- 💬 Chats
- 👤 Profile

---

## 3. Listings Management

### 3.1 My Listings
- **Tabs**: Active, Pending, Rejected, Sold, Expired
- **Cards**:
  - Pet photo, breed, price
  - Status badge (Active/Pending/Rejected)
  - Views count, inquiry count
  - Quick actions: Edit, Boost, Deactivate
- **FAB**: ➕ Create New Listing
- Search & sort options
- Empty states per tab

### 3.2 Create Listing (Multi-step)

#### Step 1: Species & Breed
- Species selector (visual cards: Dog, Cat, Bird, Fish, Exotic)
- Breed search/select (searchable dropdown)
- "Mixed Breed" option with description field

#### Step 2: Pet Details
- Name (optional, internal reference)
- Gender: Male / Female
- Age: input + unit (days/weeks/months/years)
- Color/markings: text input + color chips
- Weight: number + unit (grams/kg)
- Description: multi-line text area (min 50 chars)
- Microchip number (optional)

#### Step 3: Photos & Video
- Photo upload grid (min 3, max 10)
- Photo guidelines: "Clear, well-lit, show face + body + side"
- Video upload (optional, max 30 seconds)
- Reorder photos (drag & drop)
- Primary photo indicator (first shown)

#### Step 4: Health Information
- Vaccinated: Yes/No
- Vaccination records list:
  - Vaccine name, date, next due
  - Upload certificate photo
- Dewormed: Yes/No, last date
- Neutered/Spayed: Yes/No
- Health certificate available: toggle + upload
- Known health issues: text (optional)

#### Step 5: Pricing & Location
- Price (₹): number input
- Price type: Fixed / Negotiable
- Location: Auto-detect or manual city/area input
- Delivery option: Pickup only / Can deliver (range in km)

#### Step 6: Preview & Submit
- Full preview exactly as buyers will see
- Edit buttons per section
- "Submit for Review" button (non-verified)
- "Publish" button (verified breeders — auto-approved)
- Terms acceptance checkbox

### 3.3 Edit Listing
- Same form as create, pre-filled
- Shows current status
- "Save Changes" → re-submitted for moderation if significant changes

### 3.4 Listing Analytics
- Views over time (chart)
- Inquiries received
- Conversion rate (views → inquiries → orders)
- Competitor comparison (average for breed/city)
- "Boost Listing" CTA

### 3.5 Boost Listing
- Boost plans:
  - 3 days — ₹99
  - 7 days — ₹199
  - 14 days — ₹349
- Benefits explanation (top of search, featured section)
- Preview: how listing appears when boosted
- Payment via Razorpay

---

## 4. Orders

### 4.1 Seller Orders
- **Tabs**: New (action needed), Active, Completed, Cancelled
- **New Order Card** (highlighted):
  - Buyer info (name, city)
  - Pet ordered (photo, breed)
  - Price
  - "Accept" / "Decline" buttons
  - Timer: "Respond within 24 hours"
- **Active Order Card**:
  - Status: Accepted, Meeting Scheduled, Completed
  - Buyer chat shortcut
  - Action buttons per status

### 4.2 Order Detail (Seller View)
- Order number & date
- Pet info (photo, breed, price)
- Buyer info: Name, city (phone after acceptance)
- Order status timeline
- Chat button
- Actions:
  - New → Accept / Decline (with reason)
  - Accepted → "Coordinate Meetup" (opens chat)
  - Escrow status indicator
- Delivery/pickup instructions

### 4.3 Decline Order
- Reason selector (dropdown):
  - Pet already sold
  - Buyer seems suspicious
  - Can't arrange meetup
  - Personal reasons
  - Other (text input)
- "Confirm Decline" button
- Warning: "Frequent declines may affect your ranking"

---

## 5. Earnings & Payouts

### 5.1 Earnings Dashboard
- **Balance card**:
  - Available balance (can withdraw)
  - Pending (in escrow)
  - Total earned (lifetime)
- **Revenue chart**: Monthly earnings (6 months)
- **Recent transactions list**:
  - Order payment received
  - Payout to bank
  - Commission deducted
  - Boost payment

### 5.2 Payout History
- List of all payouts to bank
- Each: Amount, date, UTR number, status (Processed/Failed)
- Filter by month

### 5.3 Bank Account Setup
- Account holder name
- Account number (masked after save)
- IFSC code (auto-fills bank name & branch)
- Bank name (auto-filled)
- Account type: Savings / Current
- UPI ID (optional alternative)
- "Verify Account" → penny drop verification

---

## 6. Chat

### 6.1 Chat List (Seller)
- Conversations grouped: New Inquiries (unread), Active, Archived
- Each: Buyer avatar, name, listing context thumbnail, last message, time
- Quick "Accept/Decline" for new inquiries

### 6.2 Chat Room (Seller)
- Buyer name + listing context at top
- Messages with timestamps
- Quick replies: "Yes available", "When can you visit?", "Price is fixed"
- Attach: Photos, location for meetup
- "Buyer verified" / "Buyer new" indicator

---

## 7. Verification & KYC

### 7.1 KYC Submission
- Required documents based on seller type:
  - **Individual**: Aadhaar + PAN
  - **Breeder**: Aadhaar + PAN + FSSAI/Kennel registration
  - **Pet Shop**: Aadhaar + PAN + GST + Shop license
- Document upload (camera/gallery per document)
- Selfie with ID (liveness check)
- Submit for verification

### 7.2 KYC Status
- Status: Under Review / Approved / Rejected
- If rejected: Reason shown + "Resubmit" option
- Verified badge preview: "This is how buyers see your profile"

---

## 8. Breeder Features

### 8.1 Breeder Profile Setup
- Kennel/cattery name
- Registration number
- Years of experience
- Breeds specialized in (multi-select)
- Facility photos (upload gallery)
- Certifications (upload)
- Description/philosophy

### 8.2 Parent Animals Registry
- List of parent animals
- Add parent: Name, breed, age, photos, health records, achievements
- Mark as active/retired

### 8.3 Litter Management
- Record litters: Parents (link), birth date, puppy/kitten count
- Status: Available / All Sold / Reserved
- Link individual listings to litter

---

## 9. Reviews & Reputation

### 9.1 Reviews Received
- Overall rating display (stars + count)
- Rating distribution bar
- Individual review cards:
  - Buyer name, rating, date
  - Review text + photos
  - "Respond" button
- Filter by rating
- Report inappropriate review

### 9.2 Respond to Review
- Original review shown
- Response text area (max 500 chars)
- "Post Response" button
- Preview how it appears to buyers
- Note: "Be professional — response is public"

---

## 10. Settings & Account

### 10.1 Seller Profile
- Edit business name, description
- Update profile photo, cover photo
- Manage location/address
- Set auto-response message
- Set available hours

### 10.2 Performance Metrics
- Response rate (% of inquiries responded)
- Response time (average)
- Order completion rate
- Rating trend (monthly)
- Tips to improve: "Respond faster to increase visibility"

### 10.3 Notifications (Seller)
- Toggle per type:
  - New inquiries
  - New orders
  - Messages
  - Reviews
  - Payouts
  - Listing status updates
- Sound & vibration preferences

### 10.4 Settings
- Language preference
- Delete account (with confirmation)
- Switch to Buyer mode (opens customer app)
- Help & Support
- Terms, Privacy Policy
