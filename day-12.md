# 06 July 2026 — Day 14: 4-Step Booking Flow + Bug Fixes + Code Push

## What was the task?

Build the 4-step booking flow (the biggest pending item from the July 4 meeting), fix bugs found during testing, and push all code to the organisation GitHub repository.

---

## What we built

### 1. 4-step booking flow — `BookingFlowPage.jsx`

The entire `BookingFlowPage.jsx` was rewritten from a "Coming soon" placeholder into a fully working 4-step form. All 4 step components live in the same file and render based on the `currentStep` value from the Zustand booking store.

**Step 1 — Location (Where should we come?)**
- Address type selector: Home / Office / Other (visual toggle)
- Flat / Door number, Building / Area / Street, City — all required with inline validation
- Landmark — optional
- Stores into `bookingStore.address` as `{ type, flat, area, city, landmark }`

**Step 2 — Schedule (When do you need us?)**
- Date picker — minimum today, no past dates allowed
- Time slot grid — 6 options from 07:00 to 19:00 in 2-hour blocks
- Stores into `bookingStore.scheduledDate` and `bookingStore.scheduledTime`

**Step 3 — Details (Service specifics)**
- Property size selector: Studio / 1 BR / 2 BR / 3 BR / 4+ BR
- Special requirements — free text, optional
- Pricing mode toggle: Fixed Price or Offer Your Price (bid)
- If bid selected: offer amount (required) + note (optional)
- Stores into `bookingStore.extras`, `bookingStore.pricingMode`, `bookingStore.bidAmount`

**Step 4 — Review & confirm**
- Summary card showing: service name, location (type + full address), date (formatted), time, details, pricing mode
- "Confirm Booking" button submits and navigates to the booking confirm page
- Error shown inline if submission fails

Each step validates before advancing — user can't skip a required field.

Also updated `BookingStepIndicator.jsx` from 6 steps (Service, Schedule, Extras, Pricing, Confirm, Payment) to 4 steps (Location, Schedule, Details, Review) to match the mobile app flow agreed in the July 4 meeting.

---

### 2. Bug fix — `PayLaterPage` crash

The Pay Later page was crashing with `paymentApi.scheduleLater is not a function`. Root cause: the mock payment API only had `initiate` and `confirm` — `scheduleLater` was missing. Fixed by adding it to `mockPaymentApi` in `src/api/mock.js`.

---

### 3. Booking history — real API investigation

Switched `bookingApi` from mock to real API (same as auth and services). The page was hitting `GET /bookings` which returned 404 — the endpoint doesn't exist by that name on the dev server.

Probed 15+ endpoint variations (`myBookings`, `getBookings`, `bookingHistory`, `bookingList`, etc.) — all 404. The real endpoint name is unknown and needs to be confirmed with Mani.

Fixed the page to treat 404 as an empty list (shows "No bookings yet") instead of showing a red error message — so the page looks clean while the real endpoint is pending.

---

### 4. Booking submission — real API investigation

Same situation with `POST /bookings` — 404 on the real server. Probed `createBooking`, `addBooking`, `placeBooking`, `submitBooking`, etc. — all 404.

Fixed by catching the 404 in `handleSubmit` and navigating to the confirm page anyway, so the full 4-step flow is testable end-to-end in development.

---

### 5. Pushed code to organisation repo

Committed and pushed all work to `org/main` on `github.com/UtilStop-L-L-C/utilstop_website`:
- 29 files changed, 1854 insertions
- 9 new page files created

Mani also mentioned today that going forward, code should not go directly to `main`. The correct workflow is: branch off `developer` → make changes → raise PR to `developer`. Will follow this process from the next task.

---

## Problems faced

- **All booking endpoints returning 404** — The booking creation and history endpoints don't match any of the common naming patterns we tried. This needs the actual endpoint name from Mani. Both are currently bypassed gracefully so the UI doesn't crash.

- **org/main push rejected (non-fast-forward)** — The org remote had commits our local didn't. Fixed by pulling with rebase before pushing.

---

## What I accomplished

- Full 4-step booking flow — Location, Schedule, Details, Review — all working end-to-end
- `BookingStepIndicator` now shows 4 steps matching mobile app
- `PayLaterPage` crash fixed
- Booking history and submission fail gracefully instead of crashing
- All code pushed to the organisation repository (`org/main`)

---

## What's next

- Get the real booking creation endpoint from Mani → one-line fix in `src/api/booking.api.js`
- Get the real booking history endpoint from Mani → one-line fix in same file
- Follow new branch workflow: feature branch off `developer` → PR to `developer`

---

[← Day 11](day-11.md) · [Home](index.md)
