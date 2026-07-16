# Detailed Requirements & Workflows

## 1. System Overview

Multi-tenant event management and ticketing platform. It enables organizations to create/manage events, sell tickets, handle attendees, process payments, and integrate with external systems.

---

## 2. Core Entities & Relationships

- **Account**: Organization, owns events, users, settings.
- **User**: Can belong to multiple accounts, has roles (SUPERADMIN, ADMIN, ORGANIZER).
- **Event**: Belongs to an account, has products (tickets), settings, stats.
- **Product**: Ticket or add-on, linked to event, has price, capacity, rules.
- **Order**: Linked to event, user, products, payment, attendees.
- **Attendee**: Linked to order and event, has ticket info.
- **PromoCode**: Linked to event/account, applies discounts.
- **Webhook**: Configured per account/event for notifications.
- **StripePayment**: Linked to order, stores payment data.

**Entity Relationship Diagram (simplified):**
```
[Account]---<owns>---[Event]---<has>---[Product]
   |                    |                |
   |                    |                |
<has>                <has>            <has>
   |                    |                |
[User]              [Order]---<has>---[Attendee]
   |                    |
<has>                <has>
   |                    |
[Role]             [StripePayment]
```

---

## 3. Feature Requirements & Workflows

### 3.1 Event Management

**Requirements:**
- Create, update, duplicate, delete events.
- Set event metadata: title, description, location, time, images, settings.
- List/search events (public/admin), filter by date, location, organizer, status.
- View real-time/historical stats (sales, check-ins, revenue).

**Workflow: Event Creation**
```
[Organizer]--(create event form)-->[API]--(validate)-->[DB]
     |                                         |
     |<-----------success/failure---------------|
```

---

### 3.2 Ticketing & Orders

**Requirements:**
- Define products (tickets/add-ons) with price, capacity, rules.
- Cart/order creation: select products, provide attendee info.
- Validate order: capacity, promo codes, required questions.
- Checkout: Stripe/offline payment.
- Order statuses: pending, paid, cancelled, refunded, expired.
- View/download tickets/invoices.

**Workflow: Ticket Purchase**
```
[Attendee]--(select tickets)-->[Cart]--(checkout)-->[API]
     |                                         |
     |<-----------payment form------------------|
     |------------------pay-------------------->|
     |<-----------order confirmation------------|
```

---

### 3.3 Attendee Management

**Requirements:**
- Register attendees, collect custom info.
- Deliver tickets via email (PDF, QR code).
- Check-in: manual or QR scan.
- View/filter/export attendee lists.

**Workflow: Check-In**
```
[Staff]--(scan QR/manual)-->[API]--(validate)-->[DB]
     |                                         |
     |<-----------check-in result---------------|
```

---

### 3.4 User Accounts & Roles

**Requirements:**
- Register/login, password reset, invitation flow.
- Multi-account support.
- Roles: SUPERADMIN, ADMIN, ORGANIZER.
- Account settings: profile, notifications, payment.

**Workflow: User Registration**
```
[User]--(register form)-->[API]--(validate)-->[DB]
     |                                         |
     |<-----------success/failure---------------|
```

---

### 3.5 Payments & Invoicing

**Requirements:**
- Stripe integration (multiple platforms, Connect onboarding, webhooks).
- Offline/manual payments.
- Platform fees (percentage/fixed).
- Refunds (full/partial).
- Automatic invoice generation/delivery.

**Workflow: Payment**
```
[Order]--(initiate payment)-->[Stripe/Offline]
     |                             |
     |<------webhook/confirmation--|
     |                             |
[API]--(update order status)-->[DB]
```

---

### 3.6 Integrations

**Requirements:**
- Webhooks: notify external systems on key events.
- Email: transactional (orders, tickets, invites, password reset).
- Analytics: event/user/sales tracking.
- Sitemap: dynamic, paginated for SEO.

**Workflow: Webhook Notification**
```
[Event]--(trigger)-->[API]--(enqueue job)-->[Worker]
     |                                         |
     |<-----------webhook POST-----------------|
     |                                         |
[External System]<--(receive notification)-----|
```

---

### 3.7 Multi-Language & Localization

**Requirements:**
- UI/emails support multiple languages.
- Timezones/currencies per event/user.

---

## 4. Backend Workflows

**API Request Flow**
```
[Client]--(HTTP request)-->[API Router]
     |                         |
     |<--auth/validation------>|
     |                         |
[Controller]--(handler)-->[Domain Service]
     |                         |
     |<--business logic------->|
     |                         |
[Repository]<--DB read/write-->[DB]
     |                         |
[Event/Job]--(async)-->[Worker/Webhook/Email]
     |                         |
     |<--response--------------|
```

---

## 5. Frontend Workflows

**Data Fetching & State**
```
[Component]--(fetch)-->[API]
     |                   |
     |<--data/err--------|
     |                   |
[React Query]--(cache)-->[Component]
```

**Checkout Flow**
```
[User]--(select tickets)-->[Cart]
     |                         |
     |--(proceed)-->[Checkout Form]
     |                         |
     |--(submit)-->[API]--(payment)-->[Stripe]
     |                         |
     |<--confirmation----------|
```

---

## 6. Security & Compliance

- JWT authentication, refresh tokens, session management.
- Role-based access at API/UI.
- GDPR-compliant data handling, user data export/deletion.
- Audit logging, strict input validation.

---

## 7. Deployment & Configuration

- Environment variables for secrets/config.
- Docker images for deployment.
- Cloud support (DigitalOcean, AWS, etc.).
- CI/CD pipelines.

---

## 8. Assumptions

- Multi-tenancy: Each organization/account has isolated data/settings (evident from account context in code).
- Extensibility: Architecture supports new payment providers, notification channels, integrations (repository/service patterns).
- Scalability: Event-driven, background jobs for async processing (job/event code, queue config).
- Frontend/Backend Decoupling: All business logic in backend, frontend is API client (API resource structure, lack of business logic in frontend).

---

## 9. Feature Table

| Feature Area         | Backend Responsibilities                | Frontend Responsibilities                | Integration Points         |
|----------------------|-----------------------------------------|------------------------------------------|---------------------------|
| Event Management     | CRUD, validation, stats, duplication    | Event forms, listings, dashboards        | -                         |
| Ticketing/Orders     | Product mgmt, order workflow, payments  | Cart, checkout, order summary            | Stripe, Email, Webhooks   |
| Attendee Management  | Registration, check-in, ticket delivery | Registration forms, attendee lists       | Email, Webhooks           |
| User Accounts/Roles  | Auth, roles, multi-tenancy, settings    | Login, profile, team mgmt                | Email                     |
| Payments/Invoicing   | Stripe/offline, fees, refunds, invoices | Payment forms, invoice download          | Stripe, Email             |
| Integrations         | Webhooks, analytics, email, sitemaps    | Webhook config, analytics, SEO           | Webhooks, Analytics       |
| Localization         | Timezones, currencies, translations     | Locale switcher, translated UI           | -                         |

---

**If you need further breakdowns (e.g., per-entity, per-API, or more workflow diagrams), specify the area and I will expand accordingly.**