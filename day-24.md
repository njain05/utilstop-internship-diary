# 22 July 2026 — Day 24: Security Hardening — Both Repos

## What was the task?

Implement the full security hardening plan from yesterday's research across both `utilstop_apiengine` and `utilstop_website`, and get the branches pushed to GitHub so Sharanya could review.

---

## What I did

### 1. Backend security hardening — `utilstop_apiengine`

**`app.js`:**
- Added `helmet()` — sets secure HTTP headers (X-Frame-Options, X-Content-Type-Options, HSTS, etc.) in one line
- Added `express-rate-limit`:
  - `authLimiter` — 5 requests / 15 min on `/generateOTP` and `/ezp_mobile_login`
  - `otpLimiter` — 3 requests / 15 min on `/verifyOTP` and `/verify_ezp_OTP`
  - `paymentLimiter` — 10 requests / 60 sec on payment routes
- Replaced `app.use(cors())` (allows all origins) with a strict whitelist:
  ```js
  const allowedOrigins = (process.env.ALLOWED_ORIGINS || "http://localhost:5173,...").split(",")
  app.use(cors({ origin: (origin, cb) => !origin || allowedOrigins.includes(origin) ? cb(null, true) : cb(new Error("Not allowed by CORS")), credentials: true }))
  ```

**`src/common/helper.js`:**
- Added `safelyDecryptId(id)` — wraps `decrypt()` with a try/catch, returns the original value if decryption fails. Replaces the repetitive `helper.decrypt(x) ? helper.decrypt(x) : x` pattern that was scattered across controllers.

**`src/routes/payments/payment.controller.js` and `payment.service.js`:**
- Switched all `helper.decrypt(req.params.booking_id) ? ... : req.params.booking_id` patterns to `helper.safelyDecryptId(req.params.booking_id)` — cleaner and handles edge cases

**`src/routes/bookings/bookings.controller.js`:**
- Cancel booking handler: decrypt each booking ID in the incoming array before processing
- `getPayment` and `getPaymentAdmin`: added `file.user = await helper.encrypt(file.user.toString())` — user_id is now encrypted in the response alongside the already-encrypted `file.id` and `file.service_id`

### 2. Frontend security hardening — `utilstop_website`

**Moved payment keys to environment variables:**
- Stripe publishable key and Razorpay key moved out of `Constants.js` and into `.env` as `VITE_STRIPE_PUBLISHABLE_KEY` and `VITE_RAZORPAY_KEY`
- Added to `.env.example` for documentation

**Fixed `isMobile` header:**
- The `X-Is-Mobile` header was being set to the string `"false"` even when it should have been a boolean — caused backend to misdetect web requests as mobile. Fixed.

**P&M photo validation:**
- Added file type and size validation on the Packers & Movers photo upload step before sending to the API

**New `src/utils/idCrypto.js`:**
```js
import CryptoJS from 'crypto-js'
const SECRET = import.meta.env.VITE_ID_SECRET ?? ''
export function encryptId(id) { ... CryptoJS.AES.encrypt(String(id), SECRET) ... }
export function decryptId(cipher) { ... CryptoJS.AES.decrypt(cipher, SECRET) ... }
```

**Encrypt booking ID in all URL params:**
Wherever the app navigates to a route containing a `bookingId` (except `/booking/:id/details` — Sharanya confirmed it's fine to show there), the ID is now encrypted before being put in the URL:
- `BookingFlowPage.jsx` — payment navigation (×2)
- `CartPage.jsx` — checkout navigation
- `BidAcceptPage.jsx` — post-bid-accept navigation to payment
- `BidStatusPage.jsx` — bid accept link (re-encrypts after decrypting incoming param)
- `OfferPricePage.jsx` — bid status navigation
- `PaymentFlow.jsx` — booking success navigation
- `BookingDetailsPage.jsx` — booking success navigation + complete payment link
- `BookingHistoryPage.jsx` — payment and tracking links

**Decrypt on read** in destination pages:
- `PaymentPage.jsx`, `TrackingPage.jsx`, `BidStatusPage.jsx`, `BidAcceptPage.jsx` — all decrypt `useParams()` before passing the plain ID to hooks/API calls

**`vite.config.js` fix:**
- Config file was importing from `src/constants/Constants.js` which uses `import.meta.env` — unavailable at config-load time, causing a crash. Replaced with `process.env` fallback directly in the config.

### 3. Pushed branches to GitHub

- `utilstop_website` `feature/security-hardening` → pushed to `org` remote (UtilStop-L-L-C) ✅
- `utilstop_apiengine` `feature/security-hardening` → pushed to `org` remote after getting write access ✅

### 4. Sharanya review session

Sharanya reviewed the changes and raised:

1. **"Is the booking ID encrypted?"** — Confirmed: `file.id` was already encrypted in `getPayment`. The URL was the missing piece — now fixed.
2. **"Did you encrypt or decrypt?"** — Clarified: encryption was already on outgoing responses; we added decryption for incoming encrypted IDs, and now also encrypt in the frontend URL.
3. **"Check tracker page and everywhere — ID shouldn't be exposed. View booking details is okay."** — Extended the URL encryption to tracking, booking success, bid status, and bid accept routes. Booking details left plain as instructed.

---

## Problems faced

- **Write access denied on `utilstop_apiengine`** — initial push failed with 403. Asked the team; write access was granted within the session, then pushed successfully.
- **`VITE_ID_SECRET` needed to match backend `SECRET_KEY`** — frontend encryption is useless if the key doesn't match what the backend expects for decryption. Got the key from the team and added it to `.env`.
- **`vite.config.js` crash** — importing `import.meta.env` at config-load time throws because Vite hasn't set up the env object yet. Fixed by using `process.env` with a fallback.

---

## What I accomplished

- Full security hardening committed across both repos:
  - Backend: helmet, rate limiting, strict CORS, `safelyDecryptId`, user ID encryption
  - Frontend: env vars for payment keys, ID encryption in all booking-related URLs
- Both branches pushed to org GitHub
- Booking ID no longer visible as a plain integer in any URL (except /booking/details per Sharanya's instruction)

---

## What's next

- Merge `feature/security-hardening` into developer branch after Sharanya's final approval
- Continue testing the full booking → payment → tracking flow with encrypted IDs end-to-end

---

[← Day 23](day-23.md) · [Home](index.md)
