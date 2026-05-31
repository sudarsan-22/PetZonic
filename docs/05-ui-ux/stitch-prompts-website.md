# PetZonic — Stitch AI Prompts (Website)

> **Version**: 1.0.0  
> **Date**: May 31, 2026  
> **Tool**: [Stitch AI](https://stitch.ai)  
> **Framework**: Next.js 15 (App Router) — React + Tailwind CSS

---

## Brand Guidelines (Use in Every Prompt)

| Element | Value |
|---------|-------|
| Primary Color | Orange `#FF8C00` |
| Secondary Color | Teal `#14B8A6` |
| Background | White `#FFFFFF` |
| Text Color | Dark Gray `#1E293B` |
| Font (Headings) | Poppins (Bold / Semi-Bold) |
| Font (Body) | Inter (Regular / Medium) |
| Border Radius | 12px (cards), 8px (buttons) |
| Shadows | Soft elevation (`0 2px 8px rgba(0,0,0,0.08)`) |
| Style | Modern, clean, warm, pet-friendly, trustworthy |
| Logo | Orange paw print + "PetZonic" wordmark |

---

## Design Progress

| # | Page | Route | Status | Prompt # |
|---|------|-------|--------|----------|
| 1 | Homepage | `/` | ✅ Done | 1 |
| 2 | Pet Listings | `/pets` | ✅ Done | 2 |
| 3 | Pet Detail | `/pets/[slug]` | ✅ Done | 3 |
| 4 | Products | `/products` | ✅ Done | 4 |
| 5 | Product Detail | `/products/[slug]` | ✅ Done | 5 |
| 6 | Services | `/services` | ✅ Done | 6 |
| 7 | Provider Detail | `/services/[id]` | ✅ Done | 7 |
| 8 | Breed Guides | `/breeds/[breed]` | ✅ Done | 8 |
| 9 | About Us | `/about` | ✅ Done | 9 |
| 10 | Contact | `/contact` | ✅ Done | 10 |
| 11 | FAQ | `/faq` | ✅ Done | 11 |
| 12 | Legal Pages (Terms/Privacy/Refund) | `/terms`, `/privacy`, `/refund-policy` | ✅ Done | 12 |
| 13 | Privacy Policy | `/privacy` | ✅ Done (covered in #12) | — |
| 14 | Refund Policy | `/refund-policy` | ✅ Done (covered in #12) | — |
| 15 | Seller Landing | `/sell` | ✅ Done | 13 |
| 16 | Login / Signup | `/auth` | ✅ Done | 14 |
| 17 | My Account | `/account` | ✅ Done | 16 |
| 18 | My Orders | `/account/orders` | ✅ Done | 17 |
| 19 | Order Detail | `/account/orders/[id]` | ✅ Done | 18 |
| 20 | My Wishlist | `/account/wishlist` | ✅ Done | 19 |
| 21 | My Addresses | `/account/addresses` | ✅ Done (part of #16) | — |
| 22 | Account Settings | `/account/settings` | ✅ Done (part of #16) | — |
| 23 | Cart | `/cart` | ✅ Done | 20 |
| 24 | Checkout | `/checkout` | ✅ Done | 21 |
| 25 | Order Success | `/order-success` | ✅ Done | 22 |
| 26 | Search Results | `/search` | ✅ Done | 15 |
| 27 | Community Home | `/community` | ✅ Done | 25 |
| 28 | Post Detail | `/community/[id]` | ✅ Done | 26 |
| 29 | Lost & Found | `/community/lost-found` | ✅ Done | 27 |
| 30 | Blog / Education | `/learn` | ✅ Done | 28 |
| 31 | Article Detail | `/learn/[slug]` | ✅ Done | 29 |
| 32 | Seller Dashboard | `/seller` | ✅ Done | 23 |
| 33 | Seller Listings | `/seller/listings` | ✅ Done | 24 |
| 34 | Seller Orders | `/seller/orders` | ✅ Done | 30 |
| 35 | Seller Payouts | `/seller/payouts` | ✅ Done | 31 |
| 36 | Chat | `/chat` | ✅ Done | 32 |

---

## Prompts

---

### Prompt #1 — Homepage (`/`)

**Status**: ✅ Completed

```
Design a modern, warm, and inviting homepage for "PetZonic" — an Indian pet marketplace website (Next.js).

Brand: Primary Orange (#FF8C00), Secondary Teal (#14B8A6), White background, Dark Gray text (#1E293B). Fonts: Poppins for headings, Inter for body. Rounded corners (12px cards, 8px buttons), soft shadows.

Layout (top to bottom):
1. **Sticky Navbar**: Logo (orange paw + "PetZonic"), nav links (Pets, Products, Services, Community), search icon, heart (wishlist), cart icon with badge, "Login" button (orange outline), "Sell" button (orange filled). White background with subtle bottom border.

2. **Hero Section**: Full-width gradient banner (warm orange to soft peach), headline "Find Your Perfect Pet Companion" in Poppins Bold, subtext "India's most trusted pet marketplace — buy, sell & care", prominent search bar with category tabs (Pets / Products / Services), CTA button "Explore Now". Show a happy family with a golden retriever illustration/photo on the right.

3. **Category Tiles**: 7 circular/rounded-square icons in a horizontal row — Dogs, Cats, Birds, Fish, Exotic Pets, Products, Services. Each with a cute icon and label below.

4. **Featured Pets Carousel**: 6 pet cards in a horizontal scroll. Each card: pet photo (rounded top), breed name, age, gender icon, price (₹), city, verified seller badge (teal). "View All →" link.

5. **How It Works**: 3-step horizontal infographic with icons — Browse (magnifying glass) → Connect (chat bubble) → Adopt/Buy (heart + home). Clean with connecting line/arrows.

6. **Product Deals**: Section title "Today's Deals", 4-column grid of product cards with countdown timer banner at top. Each card: product image, brand, name, MRP crossed, sale price, "Add to Cart" button.

7. **Services Near You**: 3 provider cards — photo, name, service type badge, star rating, starting price, "Book" button (teal).

8. **Testimonials**: Customer review carousel with avatar, name, stars, quote text. Soft background (#FFF7ED warm cream).

9. **Download App CTA**: Split section — left side: "Get PetZonic on your phone" with App Store + Play Store buttons; right side: phone mockup showing the app.

10. **Footer**: 4-column layout — Company (About, Careers, Blog), Support (Help, FAQ, Contact), Legal (Terms, Privacy, Refund), Social (Instagram, YouTube, Twitter icons). Newsletter signup at bottom. Dark gray background (#1E293B), white text.

Style: Clean, spacious, modern e-commerce feel. Pet-friendly and trustworthy. Responsive — show desktop (1440px) version.
```

---

### Prompt #2 — Pet Listings (`/pets`)

**Status**: ✅ Completed

```
Design the Pet Listings page for PetZonic — a marketplace where users browse and filter pets for sale. Desktop view (1440px), Next.js website.

Brand: Orange (#FF8C00), Teal (#14B8A6), White BG, Dark Gray text (#1E293B). Fonts: Poppins headings, Inter body. Rounded cards (12px), soft shadows.

Use the same sticky navbar from the Homepage design.

Layout:
1. **Breadcrumb**: Home > Pets (small, subtle gray text below navbar)

2. **Page Header**: Title "Find Your Perfect Pet" (Poppins Semi-Bold, 28px), subtitle "Browse verified sellers across India". Right side: sort dropdown (Newest, Price Low→High, Price High→Low) + grid/list view toggle icons.

3. **Two-Column Layout**:

   **Left Sidebar (280px, sticky)**:
   - "Filters" heading with "Clear All" link
   - **Species**: Checkboxes — Dog, Cat, Bird, Fish, Rabbit, Exotic (with count badges)
   - **Breed**: Searchable dropdown (type to filter breeds)
   - **Price Range**: Dual-handle slider (₹0 — ₹5,00,000) with min/max input fields
   - **Gender**: Radio — Male, Female, Any
   - **Age**: Checkboxes — 0-3 months, 3-6 months, 6-12 months, 1+ year
   - **City**: Searchable dropdown with popular cities pinned (Mumbai, Delhi, Bangalore, Chennai)
   - **Verified Seller Only**: Toggle switch (teal when on)
   - Each filter section collapsible with chevron icon
   - "Apply Filters" button (orange, full-width) at bottom

   **Right Content Area (3-column card grid)**:
   - **Results count**: "Showing 1–24 of 1,847 pets"
   - **Active filter chips**: Removable pills showing applied filters (e.g., "Dog ✕", "₹5K–₹50K ✕")
   - **Pet Cards** (responsive grid):
     - Pet photo (16:9 ratio, rounded top corners)
     - Wishlist heart icon (top-right of image)
     - Verified badge overlay (teal, bottom-left of image) if applicable
     - Below image: Breed name (bold), "Age • Gender" line (gray), Price in ₹ (orange, bold), City + seller name (small gray)
     - Hover: subtle lift shadow + "View Details" overlay
   - Show 12 cards on the page

4. **Pagination**: Centered — Previous arrow, page numbers (1, 2, 3 ... 77), Next arrow. Current page highlighted in orange.

5. **No Results State**: Friendly illustration of a confused puppy, "No pets match your filters" text, "Clear Filters" button.

6. **Mobile Note**: On mobile, filters move to a slide-out drawer triggered by a "Filters" floating button.

Footer: Same as Homepage.

Vibe: Clean, browsable, easy to scan. Feels like a premium marketplace — trustworthy and organized.
```

---

### Prompt #3 — Pet Detail (`/pets/[slug]`)

**Status**: ✅ Completed

```
Design the Pet Detail page for PetZonic — the individual pet listing page where buyers see full information and contact the seller. Desktop view (1440px), Next.js.

Brand: Orange (#FF8C00), Teal (#14B8A6), White BG, Dark Gray text (#1E293B). Fonts: Poppins headings, Inter body. Rounded elements (12px), soft shadows.

Same sticky navbar as previous pages.

Layout:
1. **Breadcrumb**: Home > Pets > Golden Retriever Puppy in Bangalore

2. **Two-Column Hero Section**:

   **Left (60% width) — Image Gallery**:
   - Main image (large, 4:3 ratio, rounded 12px)
   - Thumbnail strip below (5 thumbnails, active one has orange border)
   - "Video" tab indicator if video available
   - Lightbox on click (full-screen gallery viewer)
   - Image count badge (top-left: "1/7")

   **Right (40% width) — Pet Info Card**:
   - Breed name: "Golden Retriever" (Poppins Bold, 24px)
   - Title: "Healthy Male Puppy — 8 Weeks Old"
   - Rating/trust: "Verified Seller ✓" badge (teal)
   - **Price**: ₹25,000 (orange, bold, 28px) — "Negotiable" tag if applicable
   - **Quick Info Grid** (2x3 grid of icon + label):
     - Age: 8 weeks | Gender: Male | Color: Golden
     - Weight: 2.5 kg | Vaccinated: Yes | KCI Registered: Yes
   - **CTA Buttons**:
     - "Chat with Seller" (orange filled, full-width, large)
     - "Buy Now" (teal outline, full-width, below)
     - "♡ Save to Wishlist" (text link)
   - **Seller Mini Card** (border box):
     - Seller avatar, name, rating (stars), "Member since 2024"
     - "View Seller Profile →" link
   - Location: "Indiranagar, Bangalore" with map pin icon

3. **Description Section** (full-width below):
   - "About This Pet" heading
   - Seller's paragraph description (rich text)
   - Personality traits as tags/chips: "Playful", "Friendly", "Good with Kids"

4. **Health & Vaccination Section**:
   - Table: Vaccine name | Date | Status (✓/✗)
   - Health certificate badge if uploaded
   - "Request Health Records" button

5. **Similar Listings** (horizontal scroll, 4 cards):
   - Same card style as Pet Listings page
   - "View More →" link

6. **Sticky Bottom CTA Bar** (on scroll):
   - Fixed bar with: Pet photo (small circle), breed, price, "Chat with Seller" button, "Buy Now" button
   - Appears when main CTA buttons scroll out of viewport

Footer: Same as other pages.

Vibe: Product-page feel like Amazon/Flipkart but warm and pet-friendly. Trust-focused — verified badges, health info, seller transparency. High conversion layout.
```

---

*More prompts will be added as designs progress.*
