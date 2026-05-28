# User Stories — Service Providers

> **Role**: Veterinarian / Pet Groomer / Pet Sitter / Pet Walker  
> **Description**: Professionals offering pet-related services on the platform

---

## Onboarding & Profile

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-SP01 | As a service provider, I want to register on the platform so I can offer my services | Role selection (Vet/Groomer/Sitter/Walker/Trainer), service type selection |
| US-SP02 | As a vet, I want to upload my license and qualifications so I'm verified | Vet license upload, degree certificate, registration number, issuing council |
| US-SP03 | As a service provider, I want to create my professional profile so customers find me | Profile: name, photo, bio, experience years, services offered, location/area served |
| US-SP04 | As a vet, I want to add my clinic details so customers know where to visit | Clinic name, address, map pin, photos, operating hours, parking info |
| US-SP05 | As a groomer, I want to specify services I offer with pricing | Service list: bath, haircut, nail trim, ear cleaning, etc. + price per service |
| US-SP06 | As a pet sitter, I want to specify my availability and accepted pet types | Calendar availability, pet types (dogs/cats/birds), max pets at once, home/outdoor |
| US-SP07 | As a service provider, I want my verification badge displayed so customers trust me | Verified badge on profile and listing after document approval |

---

## Availability & Scheduling

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-SP08 | As a service provider, I want to set my weekly schedule so customers book available slots | Weekly calendar: set working hours per day, break times, slot duration |
| US-SP09 | As a service provider, I want to block specific dates so I control my schedule | Block single days or date ranges, blocked dates show as unavailable |
| US-SP10 | As a service provider, I want to manage slot capacity so I'm not overbooked | Max bookings per slot, auto-disable slot when full, waitlist option |
| US-SP11 | As a vet, I want to offer both clinic visit and home visit so customers choose | Service types: In-clinic, Home visit, Online consultation (future); different pricing each |
| US-SP12 | As a groomer, I want to set different service durations so scheduling is accurate | Duration per service (bath: 30min, full grooming: 60min), buffer between appointments |

---

## Booking Management

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-SP13 | As a service provider, I want to receive booking notifications so I know my schedule | Push + SMS on new booking: customer name, service, date, time, pet details |
| US-SP14 | As a service provider, I want to confirm or reschedule a booking | Accept/suggest alternative time buttons, reason if declining |
| US-SP15 | As a service provider, I want to view my upcoming appointments in calendar view | Calendar view: today's appointments, upcoming, with customer + pet details |
| US-SP16 | As a service provider, I want to mark appointment as completed so payment is processed | "Complete" button after service, triggers review request to customer, payment processing |
| US-SP17 | As a service provider, I want to cancel a booking with advance notice | Cancel with reason, auto-notify customer, suggest rebooking |
| US-SP18 | As a service provider, I want to view booking history so I track my work | Past bookings list: date, customer, service, payment status, review received |

---

## Service Delivery

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-SP19 | As a vet, I want to access pet's medical history during consultation | View pet's vaccination records, past visit notes (if on platform), allergies |
| US-SP20 | As a vet, I want to add notes after consultation so pet owner has records | Post-visit notes: diagnosis, prescription, next visit recommendation; visible to owner |
| US-SP21 | As a groomer, I want to see pet details before appointment so I'm prepared | Pet type, breed, size, weight, any sensitivities, grooming history, special instructions |
| US-SP22 | As a pet sitter, I want to send daily updates to the pet owner so they're reassured | Photo + text update feature, push notification to owner, update history |
| US-SP23 | As a pet walker, I want to mark walk start/end so owner knows timing | Start/end buttons, duration recorded, route tracking (future), photo at end |

---

## Earnings & Payments

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-SP24 | As a service provider, I want to set my pricing so I earn fairly | Price setting per service type, home visit surcharge, combo discounts |
| US-SP25 | As a service provider, I want to see my earnings dashboard so I track income | Total earned: this week/month, pending payouts, completed payouts |
| US-SP26 | As a service provider, I want to receive weekly payouts to my bank | Bank account setup, weekly auto-payout, minimum payout threshold |
| US-SP27 | As a service provider, I want to see commission deducted so I understand my earnings | Breakdown: service fee charged, platform commission %, net earnings per booking |

---

## Reviews & Reputation

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-SP28 | As a service provider, I want to see my reviews so I know customer satisfaction | Reviews list: rating, text, customer name, date, service type |
| US-SP29 | As a service provider, I want to respond to reviews so I address feedback | Reply option per review, one reply allowed, professional tone guidelines |
| US-SP30 | As a service provider, I want to see my aggregate rating prominently | Overall rating on profile, rating breakdown (5★ to 1★), total reviews count |
| US-SP31 | As a service provider, I want to see performance metrics so I improve | Metrics: avg rating trend, repeat customers %, cancellation rate, response time |

---

## Communication

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-SP32 | As a service provider, I want to chat with customers about bookings | Chat for booking-related questions, pre-appointment coordination |
| US-SP33 | As a service provider, I want to send reminders to customers before appointments | Manual reminder option, auto-reminder 24hr before (system handles) |

---

## Account Management

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-SP34 | As a service provider, I want to update my qualifications when I get new certifications | Add new certificates, update experience, refresh profile |
| US-SP35 | As a service provider, I want to temporarily deactivate my profile when unavailable | Vacation mode: hide from search, existing bookings honored, resume date |
| US-SP36 | As a service provider, I want to contact support if I face platform issues | Support ticket: category, description, priority; response SLA shown |

---

## Telemedicine (Vet Only)

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-SP37 | As a vet, I want to offer online video consultations so I can serve remote patients | Enable telemedicine in profile, set separate pricing, set available slots |
| US-SP38 | As a vet, I want to receive a notification when a telemedicine booking is made | Push notification with patient details, symptoms, attached photos/reports |
| US-SP39 | As a vet, I want to join a video call at the scheduled time so I can consult the pet owner | "Join Call" button at scheduled time, video room with chat panel |
| US-SP40 | As a vet, I want to write a prescription after the video consultation | Prescription form: medicine name, dosage, frequency, duration, advice notes |
| US-SP41 | As a vet, I want to recommend follow-up appointments so treatment is complete | Follow-up suggestion with recommended date, auto-reminder to patient |
| US-SP42 | As a vet, I want to answer public Q&A questions so I build my reputation | Vet Q&A feed, answer questions, earn "helpful vet" badge, profile visibility boost |

---

## Educational Content Creation

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-SP43 | As a service provider, I want to upload tutorial videos so I share my expertise | Video upload: title, description, category, thumbnail; review queue before publish |
| US-SP44 | As a service provider, I want to create a multi-video course so pet owners can learn | Course builder: title, description, add lessons in sequence, set free/premium |
| US-SP45 | As a trainer, I want to see how many users enrolled in my course so I track impact | Enrollment count, completion rate, revenue from premium courses |
| US-SP46 | As a service provider, I want to write articles about pet care so I establish authority | Article editor: rich text, images, category, tags; review before publish |
