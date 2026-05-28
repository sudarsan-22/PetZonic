# PetZonic — Admin Panel Pages (Next.js)

> **Version**: 1.0.0  
> **Framework**: Next.js 15 (App Router)  
> **UI Library**: Tailwind CSS + shadcn/ui components

---

## 1. Layout

### 1.1 Sidebar Navigation
- PetZonic Admin logo
- Navigation items:
  - 📊 Dashboard
  - 👥 Users
  - 🔒 KYC Verification
  - 📋 Listing Moderation
  - 🐾 All Listings
  - 📦 Orders
  - ⚖️ Disputes
  - 🏪 Products
  - 📂 Categories
  - 🎫 Promotions
  - 🏢 Franchises
  - 💰 Revenue
  - 📈 Analytics
  - 🖼️ Banners
  - ⚙️ Settings
  - 📜 Audit Log
- Badge counts on: KYC (pending), Moderation (queue), Disputes (open)
- Collapse/expand toggle
- Admin profile avatar + role at bottom

### 1.2 Top Bar
- Page title / breadcrumb
- Global search
- Notification bell (admin alerts)
- Admin avatar + dropdown (profile, logout)

---

## 2. Pages

### 2.1 Login (`/admin/login`)
- Admin email + password
- MFA (TOTP) code input (second step)
- "Forgot password" link
- Session timeout: auto-logout after 30min inactivity

### 2.2 Dashboard (`/admin`)
- **KPI Cards Row**:
  - Today's Revenue (₹) + % change
  - New Orders count + % change
  - New Users count + % change
  - Active Listings count
- **Alerts Panel**:
  - Pending KYC: X submissions waiting
  - Moderation Queue: X listings pending
  - Open Disputes: X need resolution
- **Charts**:
  - Revenue chart (30 days, line graph)
  - Orders by type (pie: products/pets/services)
  - New users trend (bar chart, 7 days)
  - Top cities by orders (horizontal bar)
- **Quick Actions**: Moderate listings, Process KYC, View disputes
- **Recent Activity Feed**: Last 10 admin/system events

### 2.3 Users (`/admin/users`)
- **Data table** with columns:
  - Avatar, Name, Email, Phone, Role, Status, Joined, Orders, Actions
- **Filters**: Role (buyer/seller/breeder/provider), status (active/suspended/banned), KYC status, city
- **Search**: Name, email, phone
- **Bulk actions**: Suspend, send notification
- **Row actions**: View, suspend, ban, message
- **Export**: CSV download

### 2.4 User Detail (`/admin/users/[id]`)
- **Profile tab**: Full info, avatar, contact, address, role history
- **Activity tab**: Login history, actions taken
- **Orders tab**: All orders by this user
- **Listings tab**: All listings (if seller)
- **Reviews tab**: Given and received
- **Reports tab**: Reports filed against this user
- **KYC tab**: Submitted documents, verification status
- **Actions panel**: Suspend/ban/reinstate, reset password, change role, add note

### 2.5 KYC Verification (`/admin/kyc`)
- **Queue table**: Submitted date, user name, type (seller/breeder/provider), status
- **Sort**: Oldest first (FIFO)
- **Detail view** (slide-out or modal):
  - User info
  - Document images (zoomable)
  - Document type labels
  - Selfie comparison
  - Previous submissions (if resubmitted)
  - Action buttons: Approve / Reject (with reason dropdown)
- **Stats**: Approved today, rejected today, avg processing time

### 2.6 Listing Moderation (`/admin/moderation`)
- **Queue table**: Submitted date, seller, species, breed, city, photos count
- **Detail view**:
  - All photos (gallery view)
  - Video (if uploaded)
  - Pet details (species, breed, age, gender, health info)
  - Seller profile summary (rating, verification status)
  - Previous listings by this seller
  - Reported content (if flagged by users)
  - Action buttons: Approve / Reject (reason picker) / Flag for re-review
- **Keyboard shortcuts**: A = approve, R = reject, N = next
- **Batch mode**: Quick approve/reject with arrow key navigation

### 2.7 All Listings (`/admin/listings`)
- **Full table**: All listings regardless of status
- **Columns**: Photo, title, species, breed, seller, price, status, city, views, date
- **Filters**: Status, species, city, price range, date range
- **Actions**: Deactivate, feature, edit, delete

### 2.8 Orders (`/admin/orders`)
- **Data table**:
  - Order#, type (product/pet/service), buyer, seller, amount, status, date
- **Filters**: Type, status, date range, amount range
- **Row actions**: View detail, force status change, refund
- **Stats row**: Total today, processing, shipped, completed, cancelled, refunded

### 2.9 Order Detail (`/admin/orders/[id]`)
- **Order info**: Number, type, dates, status timeline
- **Items**: Product/pet details with images
- **Parties**: Buyer info, seller info (clickable → user detail)
- **Payment**: Method, Razorpay ID, amount, commission, net to seller
- **Escrow** (pet orders): Status, held amount, release/refund triggers
- **Delivery** (product orders): Shiprocket tracking, AWB, carrier
- **Chat history**: Buyer-seller conversation viewer
- **Actions**: Cancel, refund (full/partial), force complete, escalate to dispute
- **Notes**: Admin notes (internal, not visible to users)

### 2.10 Disputes (`/admin/disputes`)
- **Queue table**: Filed date, order#, issue type, buyer, seller, status
- **Priority sort**: Oldest open first
- **Detail view**:
  - Issue description (by buyer)
  - Seller response
  - Evidence: Photos/screenshots from both parties
  - Order details
  - Chat history
  - Previous disputes involving either party
  - Resolution form:
    - Decision: Favor buyer / Favor seller / Partial
    - Refund amount (editable)
    - Seller penalty: None / Warning / Suspend / Ban
    - Internal notes
    - Notify parties: toggle

### 2.11 Products (`/admin/products`)
- **Data table**: Image, name, brand, category, price, stock, status, sales
- **Filters**: Category, brand, stock status (in/out/low), active/inactive
- **Actions**: Edit, deactivate, feature, delete
- **Bulk actions**: Price update, stock update (CSV upload)
- **"Add Product" button**: Opens product form

### 2.12 Product Form (`/admin/products/new` or `/admin/products/[id]/edit`)
- **Basic info**: Name, slug (auto-generated), brand, category (tree selector)
- **Description**: Rich text editor (markdown supported)
- **Images**: Drag-and-drop upload, reorder, set primary
- **Variants**: Dynamic rows (SKU, label, price, MRP, stock, weight)
- **Specifications**: Key-value pairs (dynamic add)
- **SEO**: Meta title, meta description, Open Graph image
- **Targeting**: Species, life stage, breed size
- **Tags**: Free-form tags
- **Status**: Active / Draft / Inactive
- **Save / Publish buttons**

### 2.13 Categories (`/admin/categories`)
- **Tree view**: Draggable hierarchy (parent → child)
- **Each node**: Name, slug, icon, product count
- **Actions**: Add child, edit, delete, merge, move
- **Add/Edit form**: Name, slug, parent, icon upload, description, SEO meta

### 2.14 Promotions (`/admin/promotions`)
- **Coupons table**: Code, type (% / flat), value, min order, usage count, expiry, status
- **Create coupon form**:
  - Code (auto-generate or custom)
  - Type: Percentage / Flat amount
  - Value
  - Min order amount
  - Max discount (for %)
  - Valid from / to
  - Usage limit (total + per user)
  - Applicable: All / Specific categories / Specific products
  - Active toggle
- **Banner deals** section: Current homepage deals configuration

### 2.15 Franchises (`/admin/franchises`)
- **Applications table**: Applicant, city, status (pending/approved/active/rejected)
- **Application detail**: Business plan, investment capacity, location, experience
- **Active franchises**: Performance metrics, revenue share tracking
- **Onboarding checklist** per franchise

### 2.16 Revenue (`/admin/revenue`)
- **Summary cards**: GMV, revenue, commissions, payouts, refunds
- **Time filters**: Today, week, month, quarter, year, custom range
- **Charts**:
  - Revenue breakdown (products vs pets vs services vs commissions)
  - Daily revenue trend
  - Top selling products (table)
  - Top earning sellers (table)
- **Payout management**:
  - Pending payouts list
  - "Process Weekly Payouts" button
  - Payout history
  - Failed payouts (retry)
- **Export**: Revenue report as CSV/PDF

### 2.17 Analytics (`/admin/analytics`)
- **User analytics**: DAU, MAU, retention curve, signup funnel
- **Marketplace analytics**: Listings growth, sell-through rate, avg days to sell
- **Search analytics**: Top searches, zero-result queries, popular breeds
- **Conversion funnel**: Views → Cart → Checkout → Payment → Completion
- **Geographic**: Heatmap of orders by city
- **Cohort analysis**: User retention by signup month

### 2.18 Banners (`/admin/banners`)
- **Homepage banners**: List with image preview, link, position, active status
- **Add/edit banner**: Image upload (specs shown), link URL, display dates, position priority
- **App banners**: Same for mobile app home screen
- **Category banners**: Banners for specific category pages
- **Preview**: How it looks on desktop / mobile

### 2.19 Settings (`/admin/settings`)
- **General**: Platform name, support email, phone
- **Commission rates**: Pet sales %, product margin %, service fee %
- **Moderation rules**: Auto-approve threshold, photo minimums, listing limits
- **Delivery**: Free delivery threshold, default shipping fee, COD availability
- **Notifications**: Email templates, SMS templates
- **Payment**: Razorpay keys, payout schedule (weekly/bi-weekly)
- **Feature flags**: Enable/disable features (franchise module, services, etc.)
- **Maintenance mode**: Toggle + message

### 2.20 Audit Log (`/admin/audit-log`)
- **Table**: Timestamp, admin name, action, target type, target ID, IP
- **Filters**: Admin, action type, date range, target type
- **Detail**: Full before/after diff for data changes
- **Export**: CSV for compliance

---

## 3. Admin Roles & Access

| Page | Super Admin | Admin | Moderator | Support | Finance |
|------|:-----------:|:-----:|:---------:|:-------:|:-------:|
| Dashboard | ✅ | ✅ | ✅ | ✅ | ✅ |
| Users (full) | ✅ | ✅ | ❌ | 👁️ Read | ❌ |
| KYC | ✅ | ✅ | ✅ | ❌ | ❌ |
| Moderation | ✅ | ✅ | ✅ | ❌ | ❌ |
| Orders | ✅ | ✅ | ❌ | ✅ | 👁️ Read |
| Disputes | ✅ | ✅ | ❌ | ✅ | ❌ |
| Products | ✅ | ✅ | ❌ | ❌ | ❌ |
| Revenue | ✅ | ✅ | ❌ | ❌ | ✅ |
| Analytics | ✅ | ✅ | ❌ | ❌ | ✅ |
| Settings | ✅ | ❌ | ❌ | ❌ | ❌ |
| Audit Log | ✅ | 👁️ Read | ❌ | ❌ | ❌ |
| Franchises | ✅ | ✅ | ❌ | ❌ | ❌ |

---

## 4. Design Principles

- **Data-dense**: Maximize information per screen (tables, not cards)
- **Keyboard-first**: Shortcuts for moderation workflow
- **Real-time**: WebSocket updates for new orders, KYC submissions
- **Dark/Light mode**: Toggle in top bar
- **Responsive**: Works on tablet (for on-the-go moderation) but optimized for desktop
- **Batch operations**: Multi-select + bulk actions for efficiency
