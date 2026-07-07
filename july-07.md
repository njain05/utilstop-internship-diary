# 07 July 2026 — Day 15: Mobile App API Investigation + Booking History Fix

## What was the task?

The July 6 session ended with two booking endpoints unknown — `POST /bookings` (create) and `GET /bookings` (history) both returned 404 on the dev server. Mani shared the mobile app repo (branch 2.5) so I could read the real endpoint names from the source.

---

## What I did

### 1. Cloned and read the mobile app source

Cloned the mobile app repo (`UtilStop-L-L-C/utilstop_mobileapp`, branch 2.5) to `/Users/cheshtha/nishtha_jain/utilstop_mobileapp/`.

Key files I read:
- `src/redux/actions/main.js` — the generic `get(url)` and `post(data, url)` helpers
- `src/screens/bookings/Bookings.js` — old bookings list screen
- `src/screens/bookings/NewBookings.js` — newer bookings list screen
- `src/screens/services/otherServices/AddItems.js` — booking item creation
- `src/screens/services/otherServices/Review.js` — booking review step
- `src/screens/payments/NewPayment1.js` — payment flow

---

### 2. Found the real booking history endpoints

Two booking history endpoints exist:

| Endpoint | Method | Used in |
|---|---|---|
| `user_bookingsList` | POST | `Bookings.js` (older screen, accepts filter body `{}`) |
| `new_user_bookingsList` | GET | `NewBookings.js` (newer screen, no body) |

`POST user_bookingsList` returns `{ status: 200, data: [...array of bookings...] }`.

Each booking object has:
- `service_name` — name of the service
- `display_booking_id` — human-readable ID shown in the UI
- `booking_id` — internal ID
- `serviceDate` — date string (first 10 chars = YYYY-MM-DD)
- `service_time` — time string
- `status` — numeric code (see below)

Status code meaning:
- `1` = Pay Now (pending payment)
- `2` = Booked (waiting)
- `3` = Confirmed
- `4` = On Going
- `5` = Completed
- `6/7/8` = Cancelled
- `9` = Confirmed (alt)

---

### 3. The booking creation architecture — big discovery

The booking creation endpoint is **not a generic `/bookings` POST**. The mobile app uses a **cart-based** system where:

1. Each service has its own backend booking route, stored on the server as a field called `bookingDetailRoute` inside the `GET appData` response
2. Items are added to a cart via that service-specific route
3. After the cart is built, the user reviews it
4. Payment is initiated via `POST new_init_payment` (sends `cart_id` and amount)
5. After payment: `POST afterpaymentSuccess`
6. A single booking can contain multiple items (same cart, multiple `booking_id`s)

Other endpoints found:
- `GET other_user_bookingDetails/:bookingId` — single booking details
- `POST new_user_cancelBooking` — cancel a booking
- `GET specificCart/:cart_id` — get all items in a cart

This architecture explains why all the generic names I tried (`POST /bookings`, `POST /createBooking`, etc.) returned 404 — they simply don't exist.

---

### 4. Updated the web app with correct endpoints

**`src/api/booking.api.js`** — Updated 3 endpoints:

```js
getById: (id) => apiClient.get(`/other_user_bookingDetails/${id}`),
getHistory: (params) => apiClient.post('/user_bookingsList', params ?? {}),
cancel: (body) => apiClient.post('/new_user_cancelBooking', body),
```

**`src/pages/booking/BookingHistoryPage.jsx`** — Updated to handle the real response shape and field names:
- Status codes are now numeric (1-9) not strings like `"pending"`
- Field names: `service_name` (not `serviceName`), `display_booking_id` (not `id`), `serviceDate`, `service_time`
- Status labels and badge colours now map to numeric codes

---

## Problems faced

**`appData` requires authentication** — The `GET appData` endpoint that contains the per-service `bookingDetailRoute` values needs a Bearer token. I could not call it directly without being logged in.

**Booking creation still not directly solvable from code** — The actual route names for each service (like "Home Cleaning booking route") are stored in the database, not hardcoded in the app. To fix booking creation properly, I either need Mani to share the `bookingDetailRoute` value for each service, or the web app needs to call `appData` first and use those routes dynamically.

---

## What I accomplished

- Read the mobile app source end to end
- Found `POST user_bookingsList` — the correct booking history endpoint
- Found `GET other_user_bookingDetails/:id`, `POST new_user_cancelBooking` — booking detail + cancel
- Updated `booking.api.js` with correct endpoint names
- Updated `BookingHistoryPage.jsx` with correct response shape, numeric status codes, and real field names
- Documented the cart-based booking creation architecture

---

## What's next

- For booking creation: need to ask Mani what the `bookingDetailRoute` value is for each service (e.g. Home Cleaning, Pest Control) so we can wire up the confirm button correctly
- The 4-step booking form we built is a good UX wrapper — we just need to know the correct POST endpoint and body shape to wire the final step
- Profile edit and address book pages can be built now (endpoints like `GET endUsers/0`, `POST newUpdateUser`, `GET getAllAddress`, `POST saveAddress`)

---

[← 06 July](july-06.md) · [Home](index.md)
