# User Stories — Buyer

> **Role**: Pet Buyer / Product Customer  
> **Description**: Individual looking to buy pets, accessories, or book services

---

## Authentication & Onboarding

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-B01 | As a buyer, I want to sign up using my phone number so I can quickly create an account | OTP received in <5s, account created after verification, redirected to profile setup |
| US-B02 | As a buyer, I want to sign in with Google so I don't need to remember a password | Google OAuth flow, auto-fill name/email from Google, account linked |
| US-B03 | As a buyer, I want to set up my profile with my city so I see relevant local listings | City selection (auto-detect + manual), profile photo optional, name required |
| US-B04 | As a buyer, I want to select what pets I'm interested in so I get relevant recommendations | Pet type preference (Dog, Cat, Bird, Fish, etc.), breed preferences optional |

---

## Pet Discovery & Browsing

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-B05 | As a buyer, I want to browse pet listings near me so I can find locally available pets | Default: sorted by distance, shows distance indicator, location auto-detected |
| US-B06 | As a buyer, I want to filter pets by breed, age, and price so I find exactly what I want | Filters: species, breed, age range, price range, gender, vaccinated, location radius |
| US-B07 | As a buyer, I want to search for a specific breed so I can quickly find what I'm looking for | Type breed name → autocomplete → results; typo tolerance works |
| US-B08 | As a buyer, I want to view detailed pet information so I can make an informed decision | Full details: breed, age, gender, weight, vaccination status, health records, photos, seller info |
| US-B09 | As a buyer, I want to see the pet's vaccination records so I know it's healthy | View uploaded vaccination certificates, checklist of completed vaccines |
| US-B10 | As a buyer, I want to save pets to my wishlist so I can compare them later | Heart icon to save, wishlist page shows all saved, remove option |
| US-B11 | As a buyer, I want to see similar pets so I have more options | "Similar pets" section on listing page, based on breed + price + location |
| US-B12 | As a buyer, I want to get notified when a pet matching my criteria is listed | Save search filters → get push notification on new matching listings |

---

## Pet Purchase

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-B13 | As a buyer, I want to chat with the seller before buying so I can ask questions | "Chat with Seller" button → opens real-time chat, can ask about pet |
| US-B14 | As a buyer, I want to negotiate the price in chat so I can get a better deal | Price discussion in chat, seller can update listing price or accept offer |
| US-B15 | As a buyer, I want to make a secure payment for a pet so my money is protected | Escrow: money held until I confirm receipt, multiple payment options |
| US-B16 | As a buyer, I want to coordinate a meetup with the seller so I can see the pet in person | Share location in chat, meetup point suggestions, date/time coordination |
| US-B17 | As a buyer, I want to confirm receipt of the pet so the seller gets paid | "Confirm Receipt" button in order, triggers payment release to seller |
| US-B18 | As a buyer, I want a health guarantee period so I can return the pet if it's sick | 7-day health guarantee, raise issue if pet is unhealthy, dispute resolution |

---

## Product Shopping

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-B19 | As a buyer, I want to browse pet products by category so I find what I need | Categories: Food, Accessories, Toys, Grooming, Health, Housing; visual cards |
| US-B20 | As a buyer, I want to add products to cart so I can buy multiple items | "Add to Cart" button, cart badge updates, continue shopping flow |
| US-B21 | As a buyer, I want to select product variants (size/flavor) so I get the right item | Variant selector on product page, price updates per variant, stock shown |
| US-B22 | As a buyer, I want to apply a coupon code so I get a discount | Coupon input field at checkout, instant discount calculation, error for invalid codes |
| US-B23 | As a buyer, I want to choose delivery time so I receive it when convenient | Available slots displayed, express vs standard option, estimated delivery date |
| US-B24 | As a buyer, I want to pay via UPI so I can use my preferred payment method | UPI intent (GPay, PhonePe), UPI ID entry, QR code option |
| US-B25 | As a buyer, I want to track my order so I know when it will arrive | Order status timeline, tracking link, push notification on status change |
| US-B26 | As a buyer, I want to return a product if it's defective so I get my money back | "Return" button within 7 days, select reason, schedule pickup, auto-refund |

---

## Services

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-B27 | As a buyer, I want to find vets near me so I can take my pet for checkup | Vet listing by distance, filter by specialization, show ratings |
| US-B28 | As a buyer, I want to book a vet appointment so I don't have to wait | Select vet → choose service → pick date/time slot → confirm booking |
| US-B29 | As a buyer, I want to book a pet grooming session so my pet stays clean | Select grooming service → choose home visit/salon → pick slot → book |
| US-B30 | As a buyer, I want to find a pet sitter when I travel so my pet is cared for | Pet sitter listings, date range selection, pet details shared, booking confirmed |
| US-B31 | As a buyer, I want to get reminders for my vet appointment so I don't forget | Push notification 24hr + 1hr before appointment |

---

## Reviews & Feedback

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-B32 | As a buyer, I want to read reviews of sellers so I can trust them | Seller profile shows reviews, overall rating, individual review text/stars |
| US-B33 | As a buyer, I want to write a review after purchase so others know my experience | Review form: 1-5 stars, text, optional photos; appears after delivery confirmed |
| US-B34 | As a buyer, I want to read product reviews so I buy quality items | Reviews on product page, sorted by helpfulness, verified purchase badge |
| US-B35 | As a buyer, I want to rate a service provider so others can benefit | Post-service rating prompt, criteria: punctuality, quality, friendliness |

---

## Account Management

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-B36 | As a buyer, I want to manage my addresses so checkout is faster | Add/edit/delete addresses, set default, pincode auto-fill city/state |
| US-B37 | As a buyer, I want to view my order history so I can track past purchases | List of all orders, filter by status, order details expandable |
| US-B38 | As a buyer, I want to manage notification preferences so I only get relevant alerts | Toggle per category: Orders, Chat, Promotions, Pet alerts |
| US-B39 | As a buyer, I want to contact customer support if I have an issue | In-app support: FAQ section, raise ticket, chat with support (future) |
| US-B40 | As a buyer, I want to delete my account if I no longer use the app | Delete account option, 30-day grace period warning, data deletion confirmation |

---

## Community & Forums

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-B41 | As a buyer, I want to browse community forums so I can learn from other pet owners | Forum categories visible, trending posts on top, search within forums |
| US-B42 | As a buyer, I want to create a post in the forum so I can ask questions about my pet | Create post with title, body, images, category, tags; post appears after submission |
| US-B43 | As a buyer, I want to reply to forum posts so I can share my experience | Reply form under each post, nested replies (max 2 levels), @mention support |
| US-B44 | As a buyer, I want to upvote/downvote posts and replies so the best content surfaces | Vote buttons, karma visible, most-voted answers rise to top |
| US-B45 | As a buyer, I want to follow a post so I get notified of new replies | Follow toggle, notifications on new replies, unfollow anytime |
| US-B46 | As a buyer, I want to report a lost pet so nearby users can help me find it | Lost pet form: photos, last seen location, pet details; nearby users notified |
| US-B47 | As a buyer, I want to browse lost & found posts near me so I can help find lost pets | Map view + list view, filter by species/city/radius, "I spotted this pet" button |

---

## Education & Training

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-B48 | As a buyer, I want to watch tutorial videos about pet care so I can be a better pet parent | Video library with categories, search, filter by species/topic, play in-app |
| US-B49 | As a buyer, I want to read articles about pet health so I stay informed | Article list with rich content, images, related articles, bookmarkable |
| US-B50 | As a buyer, I want to enroll in a training course so I can learn step-by-step | Course catalog, enroll button, track progress, video lessons in sequence |
| US-B51 | As a buyer, I want to book an online vet consultation so I can get quick advice | Select vet, pick date/time slot, describe symptoms, pay, get video link |
| US-B52 | As a buyer, I want to join a video call with the vet at the scheduled time | Video call screen, screen share, chat alongside, prescription received after |
| US-B53 | As a buyer, I want to ask a question in vet Q&A so I get expert advice for free | Public Q&A section, ask question, vet answers within 24h, visible to all |
| US-B54 | As a buyer, I want to use the feeding calculator so I know how much to feed my pet | Input pet details (species, breed, age, weight, activity) → get daily food recommendation |
| US-B55 | As a buyer, I want to view breed encyclopedia so I understand my pet's characteristics | Breed pages with info: temperament, care needs, lifespan, common health issues |

---

## Pet Insurance

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-B56 | As a buyer, I want to browse pet insurance plans so I can protect my pet | Plans listed with coverage, premium, partner logo; filter by species/coverage type |
| US-B57 | As a buyer, I want to compare insurance plans side by side so I pick the best one | Select 2-3 plans → comparison table with coverage details, exclusions, price |
| US-B58 | As a buyer, I want to calculate my premium based on my pet's details so I know the cost | Input breed, age, gender → see premium quotes from multiple partners |
| US-B59 | As a buyer, I want to purchase an insurance policy online so my pet is covered | Select plan, enter pet details, pay premium via Razorpay, receive policy PDF |
| US-B60 | As a buyer, I want to view my active policies so I know what's covered | My Policies section showing active/expired policies with details |
| US-B61 | As a buyer, I want to file an insurance claim when my pet needs treatment | Claim form: incident details, upload vet bills/reports, submit; track status |
| US-B62 | As a buyer, I want to track my claim status so I know when I'll get reimbursed | Claim timeline: Submitted → Under Review → Approved/Rejected → Paid |
| US-B63 | As a buyer, I want insurance recommendations after buying a pet so I can protect it immediately | Post-purchase insurance suggestion card, quick-enroll flow |
