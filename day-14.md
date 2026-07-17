# 08 July 2026 — Day 14: Team PR Merged, Payment Flow Built, Maps & Stripe Integrated

## What was the task?

A big PR came in from the team (Sharanya) that added the full Packers & Movers booking flow, country selection modal, and Stripe payment infrastructure. The task was to review it, merge it, and get everything working — env config, payment routing, and the Maps integration were all broken out of the box.

---

## What I did

### 1. Pulled in the team's `features` branch (PR #1)

The organisation repo's `features` branch landed as a pull request. It was a large chunk of work:

**New components:**
- `BookingStepIndicator.jsx` — step tracker UI for the booking flow
- `CountrySelectModal.jsx` — modal for switching between UAE and India
- `PaymentFlow.jsx` — full Stripe payment form + order summary UI
- `StripePaymentForm.jsx` — Stripe Elements wrapper
- `packersMovers/LocationPickerModal.jsx` — Google Maps-based pickup/drop location picker
- `packersMovers/SavedAddressList.jsx` — shows saved addresses from user profile
- `packersMovers/StepAddItems.jsx` — inventory step for Packers flow
- `packersMovers/StepMovingDetails.jsx` — date, floor, and van selection
- `packersMovers/StepPackersReview.jsx` — review and confirm step

**New hooks:**
- `useBooking.js` — encapsulates booking creation logic and state
- `usePaymentSummary.js` — fetches and formats payment breakdown from API

**New utilities:**
- `src/utils/googleMaps.js` — loads Google Maps JS API and exports geocoding helpers
- `src/utils/stripeClient.js` — initialises the Stripe.js client using the env key

The PR touched 30 files with 1378 insertions. Merged into `nishtha-dev` after reviewing.

---

### 2. Env config and Constants.js wrestling

After merging, the app had runtime errors. Traced them to two issues:

**Missing env vars:** The new Google Maps and Stripe integrations needed `VITE_GOOGLE_MAPS_API_KEY`, `VITE_STRIPE_PUBLISHABLE_KEY`, and `VITE_GOOGLE_PLACES_API_KEY` set in `.env`. These were added to `Constants.js` hardcoded as strings (to keep the app runnable without the real keys), and `env.js` was updated to expose them via the `ENV` object.

**Vite config proxy issue:** The `vite.config.js` proxy rewrite rules needed adjustment after the new endpoints were added. Updated the rewrite logic to ensure the `/api` prefix is correctly stripped before forwarding to the backend.

---

### 3. Payment page overhaul

The `PaymentFlow.jsx` that came in the PR was skeletal — roughly 140 lines. After testing, it needed significant work:

- **Order summary panel** — Shows service name, date, time slot, offering, and price breakdown. Pulls from the booking store, not a new API call.
- **Stripe Elements integration** — The card input form was wired up properly with `useStripe()` and `useElements()` hooks. Handles loading state, error messages from Stripe, and the `paymentIntent.confirm` step.
- **PaymentPage.jsx** — Added a wrapper page that decides whether to show Pay Now (Stripe) or Pay Later (confirmation screen). Routes updated in `AppRouter.jsx` and `routes.js`.
- **LocationPickerModal cleanup** — The map initialisation had a race condition where the map container wasn't in the DOM when `new google.maps.Map()` was called. Added a `useEffect` with a ref check to defer initialisation.

End of day: Pay Now flow goes all the way through to a Stripe confirmation screen. Pay Later shows a booking reference. Both are integrated with the router correctly.

---

## Problems faced

- **Runtime crashes after merge** — Missing `.env` keys caused `undefined` errors in Google Maps and Stripe init. Fixed by adding temp values to `Constants.js`.
- **Vite proxy double-path** — Proxy was forwarding to `/api/api/v1/...`. Fixed by updating rewrite target.
- **LocationPickerModal map race condition** — `new google.maps.Map()` called before the container div was in the DOM. Fixed with a `useEffect` ref guard.

---

## What I accomplished

- Reviewed and merged PR #1 (Packers & Movers flow + payment infrastructure) from the team
- Integrated Google Maps utility and Stripe client setup
- Fixed env variable setup for Maps and Stripe keys
- Resolved Vite proxy rewrite issues
- Built out PaymentFlow.jsx properly — Stripe Elements form, order summary, loading and error states
- Added PaymentPage.jsx as the unified payment entry point
- Fixed LocationPickerModal map initialisation race condition
- Updated routing so booking confirmation → payment page works end to end

---

## What's next

- Packers & Movers flow still needs testing with real addresses (Google Maps API key required)
- Profile edit page (`GET endUsers/0`, `POST newUpdateUser`)
- Address book page (`GET getAllAddress`, `POST saveAddress`)
- Security audit of Constants.js — currently has keys hardcoded (need to move to `.env`)

---

[← Day 13](day-13.md) · [Home](index.md) · [Day 15 →](day-15.md)
