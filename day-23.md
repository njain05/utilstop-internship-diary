# 21 July 2026 — Day 23: Security Research and Backend Deep-Dive

## What was the task?

Full day of research and planning before writing any code. Needed to understand exactly how the backend handles IDs today, what the mobile app expects, and what we can safely change without breaking existing behaviour.

---

## What I did

### 1. Backend code audit — `utilstop_apiengine`

Read through `src/routes/bookings/bookings.controller.js`, `src/routes/payments/payment.controller.js`, `src/routes/payments/payment.service.js`, and `src/common/helper.js` to understand the encryption situation.

**Findings:**
- `helper.encrypt(str)` / `helper.decrypt(ciphertext)` — CryptoJS AES with `process.env.SECRET_KEY` — already exists and is used in places like ad IDs and some booking list responses.
- `getPayment` already encrypts `file.id` and `file.service_id` in the response — but `file.user` (the user_id on the booking) is sent plain.
- Payment controller uses `helper.decrypt(req.params.booking_id)` when receiving IDs — this only works if the client sends an encrypted ID. If the client sends a plain integer (which the website currently does), `decrypt()` returns falsy and it falls back to the raw param.
- No `safelyDecryptId` wrapper existed — every call either decrypts or falls back manually, inconsistently.

**Planned fix:**
- Add a `safelyDecryptId` wrapper to `helper.js` that tries decryption and returns the raw value on failure — removes the repetitive ternary across controllers.
- Add `helmet` for HTTP security headers.
- Add `express-rate-limit` for auth/OTP/payment endpoints.
- Lock down `cors()` — replace the open wildcard with a whitelist from `ALLOWED_ORIGINS` env var.
- Encrypt `file.user` in `getPayment` and `getPaymentAdmin` responses.

### 2. Frontend audit — `utilstop_website`

Traced every place in the React app that navigates to a route with a `bookingId` in the URL:

| Route | Places that navigate |
|-------|---------------------|
| `/payment/:bookingId` | `BookingFlowPage` (×2), `CartPage`, `BidAcceptPage`, `BookingDetailsPage`, `BookingHistoryPage` |
| `/tracking/:bookingId` | `BookingHistoryPage` |
| `/booking/:bookingId/success` | `PaymentFlow`, `BookingDetailsPage` |
| `/booking/bids/:bookingId` | `OfferPricePage` |
| `/booking/bids/:bookingId/accept` | `BidStatusPage` |
| `/booking/:bookingId/details` | leave plain — visible booking reference is fine |

Plan: add a `src/utils/idCrypto.js` utility (CryptoJS AES, same key via `VITE_ID_SECRET`) with `encryptId` / `decryptId`, and apply it at every entry and exit point.

### 3. Mobile compatibility check

Reviewed how mobile currently calls `getPayment` — it sends the booking ID from its own in-memory state (not from a URL param), so the mobile always sends plain integers. The backend's `safelyDecryptId` fallback handles both — no mobile impact.

The `file.user` encryption is safe because the mobile app reads user info from the JWT / auth state, not from the `getPayment` response's `user` field.

---

## Problems faced

- CryptoJS AES produces a base64 string that contains `+`, `/`, `=` characters which are not URL-safe. Will need to handle this in `encryptId` — React Router encodes the param automatically via `buildRoute`, so the URL will be percent-encoded, but the `decryptId` function will need to handle the decoded value correctly.

---

## What I accomplished

- Full audit of backend encryption patterns across both repos
- Mapped every frontend navigation point that exposes a plain booking ID in the URL
- Confirmed mobile compatibility — no changes needed on mobile side
- Detailed implementation plan ready for tomorrow

---

## What's next

- Implement all security hardening changes in both repos and push branches

---

[← Day 22](day-22.md) · [Home](index.md) · [Day 24 →](day-24.md)
