# PetZonic — User Flows

> **Version**: 1.0.0  
> **Date**: May 28, 2026

---

## 1. Buyer: Browse & Purchase Pet

```mermaid
flowchart TD
    A[Open App] --> B[Homepage]
    B --> C{Browse or Search?}
    C -->|Browse| D[Browse by Category/Species]
    C -->|Search| E[Search bar → Type breed]
    D --> F[Pet Listings Grid]
    E --> F
    F --> G[Apply Filters]
    G --> F
    F --> H[Tap Pet Card]
    H --> I[Pet Detail Page]
    I --> J{Interested?}
    J -->|Save| K[Add to Wishlist]
    J -->|Chat| L[Chat with Seller]
    J -->|Buy| M[Initiate Purchase]
    L --> N[Negotiate/Ask Questions]
    N --> M
    M --> O[Select Payment Method]
    O --> P[Pay via Razorpay]
    P --> Q{Payment Success?}
    Q -->|Yes| R[Order Confirmed - Escrow Held]
    Q -->|No| S[Retry Payment]
    S --> O
    R --> T[Coordinate Meetup via Chat]
    T --> U[Meet Seller - Receive Pet]
    U --> V[Confirm Receipt in App]
    V --> W[Escrow Released to Seller]
    W --> X[Rate Seller & Pet]
```

---

## 2. Buyer: Purchase Products

```mermaid
flowchart TD
    A[Homepage → Products Section] --> B[Browse Categories]
    B --> C[Product Listing Page]
    C --> D[Tap Product]
    D --> E[Product Detail Page]
    E --> F[Select Variant - Size/Flavor]
    F --> G[Add to Cart]
    G --> H{Continue Shopping?}
    H -->|Yes| C
    H -->|No| I[View Cart]
    I --> J[Apply Coupon Code]
    J --> K[Proceed to Checkout]
    K --> L[Select/Add Address]
    L --> M[Choose Delivery Slot]
    M --> N[Review Order Summary]
    N --> O[Select Payment]
    O --> P[Pay]
    P --> Q[Order Confirmed]
    Q --> R[Track Order]
    R --> S[Order Delivered]
    S --> T[Rate & Review Product]
```

---

## 3. Seller: Create Pet Listing

```mermaid
flowchart TD
    A[Open Seller App] --> B[Dashboard]
    B --> C[Tap 'Create Listing']
    C --> D[Select Species]
    D --> E[Select Breed]
    E --> F[Enter Pet Details]
    F --> G[Upload Photos - Min 3]
    G --> H[Upload Video - Optional]
    H --> I[Add Health Info & Vaccinations]
    I --> J[Set Price & Price Type]
    J --> K[Set Location]
    K --> L[Preview Listing]
    L --> M{Satisfied?}
    M -->|Edit| F
    M -->|Submit| N{Seller Verified?}
    N -->|Yes - Breeder| O[Auto Approved → ACTIVE]
    N -->|No| P[Submitted for Review]
    P --> Q[Admin Reviews]
    Q --> R{Approved?}
    R -->|Yes| O
    R -->|No| S[Rejected - Reason Shown]
    S --> F
    O --> T[Listing Live - Receive Inquiries]
```

---

## 4. Seller: Handle Pet Order

```mermaid
flowchart TD
    A[Receive Order Notification] --> B[View Order Details]
    B --> C{Accept or Decline?}
    C -->|Decline| D[Provide Reason → Order Cancelled → Buyer Refunded]
    C -->|Accept| E[Order Accepted]
    E --> F[Chat with Buyer]
    F --> G[Coordinate Meetup]
    G --> H[Meet Buyer - Hand Over Pet]
    H --> I[Buyer Confirms Receipt]
    I --> J[Escrow Released]
    J --> K[Payment in Next Weekly Payout]
    K --> L[Receive Review from Buyer]
```

---

## 5. Buyer: Book Vet Appointment

```mermaid
flowchart TD
    A[Services Tab] --> B[Select 'Veterinary']
    B --> C[Vet Listing - Sorted by Distance]
    C --> D[View Vet Profile]
    D --> E[Select Service Type]
    E --> F[Choose Date]
    F --> G[Pick Time Slot]
    G --> H[Add Pet Details & Notes]
    H --> I[Review Booking]
    I --> J[Confirm & Pay]
    J --> K[Booking Confirmed]
    K --> L[Reminder 24hr Before]
    L --> M[Attend Appointment]
    M --> N[Vet Adds Notes]
    N --> O[Rate & Review Vet]
```

---

## 6. New User: Onboarding

```mermaid
flowchart TD
    A[Download App] --> B[Welcome Screen]
    B --> C[Enter Phone Number]
    C --> D[Receive OTP]
    D --> E[Verify OTP]
    E --> F{New User?}
    F -->|Yes| G[Profile Setup]
    F -->|No| H[Home Screen]
    G --> I[Enter Name]
    I --> J[Select City]
    J --> K[What pets interest you?]
    K --> L{Want to sell?}
    L -->|No| H
    L -->|Yes| M[Select Role - Seller/Breeder]
    M --> N[KYC Prompt - Can do later]
    N --> H
```

---

## 7. Admin: Moderate Listing

```mermaid
flowchart TD
    A[Admin Panel - Dashboard] --> B[Moderation Queue Badge]
    B --> C[Pending Listings List]
    C --> D[Open Listing for Review]
    D --> E[Check Photos]
    E --> F[Verify Details]
    F --> G[Check Seller Profile]
    G --> H{Decision}
    H -->|Approve| I[Listing Goes Live]
    H -->|Reject| J[Select Rejection Reason]
    J --> K[Seller Notified with Reason]
    I --> L[Next in Queue]
    K --> L
```

---

## 8. Dispute Resolution Flow

```mermaid
flowchart TD
    A[Buyer Raises Issue] --> B[Select Issue Type]
    B --> C[Upload Evidence - Photos/Screenshots]
    C --> D[Dispute Created]
    D --> E[Admin Notified]
    E --> F[Admin Reviews Both Sides]
    F --> G[Contact Seller for Response]
    G --> H[Seller Provides Evidence]
    H --> I{Admin Decision}
    I -->|Favor Buyer| J[Refund from Escrow]
    I -->|Favor Seller| K[Release Escrow to Seller]
    I -->|Partial| L[Partial Refund]
    J --> M[Both Parties Notified]
    K --> M
    L --> M
```
