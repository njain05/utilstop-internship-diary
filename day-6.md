# 29 June 2026 — Day 6: Service Pages, Booking & Bid Flow, Payments, Account Pages, Layout, Mock API

## What was the task?
Build the entire core user journey (service browsing → booking → bidding → payment) and the full account section in one big push. Also build the app layout (header + footer), the tracking page, the mock API system, and wire everything through the router.

---

## Part 1 — Service Pages, Booking, Bid & Payment

### Service Catalogue data (`src/services/`)
Defined all 6 services as structured data before building any UI:
```js
{
  id: 'home-cleaning',
  slug: 'home-cleaning',
  displayName: 'Home Cleaning',
  description: '...',
  supportsBidding: true,
  countries: ['UAE', 'IN'],
  configModule: () => import('./homeCleaning'),
}
```
Files: `homeCleaning.js`, `furnitureCare.js`, `homeMaintenance.js`, `logistics.js`, `pestControl.js`, `personalCare.js`

### Service List Page (`src/pages/services/ServiceListPage.jsx`)
- Renders all 6 service cards in a responsive grid (1 col → 2 col → 3 col)

### Service Card Component (`src/components/service/ServiceCard.jsx`)
- Icon, name, description, "Offer Your Price" badge if bidding is supported
- Links to the service detail page

### Service Detail Page (`src/pages/services/ServiceDetailPage.jsx`)
- Reads `:serviceSlug` from URL params
- Shows service info, pricing tiers, feature list
- "Book Now" CTA leads into the booking flow

### Booking Flow Page (`src/pages/booking/BookingFlowPage.jsx`)
Multi-step wizard using `useBooking` hook:
- `BookingStepIndicator` component shows progress
- Steps: Select Date → Select Time → Enter Address → Requirements → Review → Confirm
- Back/Next navigation between steps
- **Step content is a placeholder** — structure and navigation work but forms inside each step depend on real API data (available slots, address lookup) not available yet

### Booking Confirm / Success Pages
- `BookingConfirmPage.jsx` — shows summary before final submission
- `BookingSuccessPage.jsx` — confirmation with booking ID

### Booking History Page (`src/pages/booking/BookingHistoryPage.jsx`)
- Fetches bookings via `useBooking` hook
- Status badges: In Progress (orange), Completed (green), Cancelled (red)

### Bid Flow (`src/pages/bid/`)
- `OfferPricePage.jsx` — bid amount input, Zod validated (number, 10–99,999), DOMPurify sanitized
- `BidStatusPage.jsx` — polling/waiting screen for vendor responses
- `BidAcceptPage.jsx` — shows a specific vendor bid to accept or reject

### Payment Pages (`src/pages/payment/`)
- `PayNowPage.jsx` — payment form, initiates gateway redirect
- `PayLaterPage.jsx` — confirms payment is deferred to service start

### BidForm Component (`src/components/forms/BidForm.jsx`)
- Reusable bid amount form with Zod validation + DOMPurify sanitization

---

## Part 2 — Layout, Account Pages, Tracking, Mock API

### Layout Components (`src/components/layout/`)

**Header (`Header.jsx`)**
- Desktop: logo left, nav center, auth buttons right
- Logged in: shows user name, "My Bookings" link, Logout button
- Reads auth state directly from Zustand — no prop drilling
- Mobile: hamburger menu

**Footer (`Footer.jsx`)**
- 4-column grid: brand + tagline, Services links, Company links, Contact
- Responsive stacking on mobile

**PageWrapper (`PageWrapper.jsx`)**
- Wraps every page with Header + Footer using React Router's `<Outlet />`
- Full-width — each page section manages its own `container-pad`

### Account Pages (`src/pages/account/`)
- `DashboardPage.jsx` — "Hello, {name}" greeting + quick-action cards to all account sections
- `ProfilePage.jsx` — user info display + edit form with React Hook Form + Zod
- `AddressBookPage.jsx` — placeholder (real CRUD needs backend)
- `NotificationsPage.jsx` — placeholder (needs backend push/websocket)

### Tracking Page (`src/pages/tracking/TrackingPage.jsx`)
- Shows technician name, ETA, status from `useTracking` hook (polls every 15s)
- Map placeholder — real integration (Google Maps / Mapbox) to be added later

### Hooks (`src/hooks/`)
All 5 TanStack Query data-fetching hooks:
- `useAuth.js` — login, register, logout, refreshSession
- `useBooking.js` — booking history, create booking, step navigation
- `useServices.js` — all services, service by slug
- `useBid.js` — submit bid, get bid status, accept bid
- `useTracking.js` — get tracking data for a booking

### Mock API (`src/api/mock.js`)
Complete in-memory mock for every endpoint:
- `mockAuthApi` — login always succeeds, register stores user in memory, refreshToken returns a fake JWT
- `mockBookingApi` — 3 sample bookings: In Progress, Completed, Cancelled
- All other mocks return plausible data with 200–600ms simulated delay
- Activated by `VITE_MOCK_API=true` — no backend needed to run the full UI

### Router wired up (`src/routes/AppRouter.jsx`)
Every page connected to its route:
- Public routes under `PageWrapper`
- Auth routes under `PublicOnlyRoute`
- Protected routes under `PrivateRoute`
- Account sub-routes nested under `/account`
- 404 catch-all at `*`

### Utilities
- `formatCurrency.js` — AED (UAE) or INR (India) based on selected country
- `formatDate.js` — consistent date/time formatting
- `constants.js` — booking statuses, countries, city lists

---

## Problems faced

- The `useTracking` polling interval: too short hammers the API, too long makes ETA look stale. Set to 15 seconds for now
- The booking flow's step content is placeholder — it depends on real API responses (available time slots, address lookup) that don't exist yet. The step navigation works but the forms are empty
- The bid amount field is security-critical: user-typed text going into an API. Applied DOMPurify + Zod double protection
- `BookingHistoryPage` and `ProfilePage` are minimal — noted as next things to flesh out

---

## What I accomplished

- Service catalogue data for all 6 services
- Service List, Service Detail, full Booking Flow, Bid flow (3 pages), both Payment pages
- Full app layout — Header, Footer, PageWrapper
- All 4 Account pages
- Tracking page with polling hook
- All 5 TanStack Query hooks
- Complete mock API covering every endpoint
- Every single page connected to the router
- Utility functions for currency, dates, constants

---

[← Day 5](day-5.md) · [Home](index.md) · [Day 7 →](day-7.md)
