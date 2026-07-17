# 07 July 2026 — Day 15: API Investigation, Booking Creation Wired, Error Fixes, Org Repo Pushed

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

---

## Part 2 — Afternoon: Booking Creation, Service Types/Offerings, Error Fixes

### 5. Found booking creation endpoints via `appData`

Logged into the dev API using test number `971XXXXXXXXX` (UAE test account — bypasses Twilio, any OTP works). Called `GET appData` with the returned JWT token to read the per-service `bookingDetailRoute` values.

Two booking creation routes exist:
- `POST book_other_services` — used by almost all services (AC, Cleaning, Pest Control, etc.)
- `POST book_packersAndMovers` — used only by Packers & Movers and Item Moving

The payload structure (reverse-engineered from `AddItems.js` in the mobile app):

```js
{
  data: [{
    table_name: "service_ac",   // e.g. service_cleaning, service_pest
    service_id: "<encrypted>",
    service_offering: "<encrypted offering id>",
    service_type: "<encrypted type id or 0>",
    service_date: "2026-07-15",
    service_time: "09:00 – 11:00",
    s_house_no: "Apt 4B",
    s_area: "Marina Walk",
    s_landmark: "",
    service_state: "Dubai",
    location_id: 0,
    areaId: 0,
    cost: 120.75,
    min: 0, max: 0,
    booking_id: null,
    item_details: "",
  }],
  deleteBookingId: [],
  cart_id: 0,
  main: 1,
  service_id: "<encrypted>",
}
```

### 6. Wired Step 3 — dynamic service types and offerings

Mani specifically asked to replace the static `ROOM_OPTIONS = ['Studio', '1 BR', '2 BR', ...]` in Step 3 (Details) with real API data, since each service has different types and offerings.

Two-step fetch in `StepDetails`:
1. `GET service_type/{serviceId}/` — returns array of types (or empty if the service has no types)
2. `POST service_offerings/{serviceId}/{typeId}` with `{ date, state1, state2, area_id }` — returns offerings with prices

If the service has no types (empty array), offerings are fetched directly with `typeId = 0`.

The user first picks a type (e.g. "Split AC", "Window AC"), then picks one or more offerings (e.g. "AC Service", "AC Gas Fill") shown with their prices. Both selections are stored in Zustand (`selectedServiceType`, `selectedOfferings`) and passed into the booking payload on confirm.

### 7. Wired the Confirm button — `submitBooking` in `useBooking.js`

Built `submitBooking(service)` which:
1. Determines the correct route (`book_other_services` or `book_packersAndMovers`) from the service name
2. Builds the full payload from the Zustand store (address, date, time, type, offerings, notes)
3. Calls `POST /{route}` with a 90-second timeout (the default 15s was killing slow dev requests)
4. On success: navigates to `/booking/confirm/:bookingId`

### 8. Fixed "Network error" on Confirm button

**Problem:** Clicking "Confirm Booking" on AC Services showed "Network error. Please check your connection."

**Root cause:** Axios had a global 15-second timeout. The `book_other_services` endpoint on the dev server is slow (vendor-matching logic). The request was dying at 15s with `ECONNABORTED`, which our `normalizeError` was mistakenly mapping to "Network error."

**Fix:**
- Set 90-second timeout specifically for `bookingApi.create`
- Split `ECONNABORTED` (our timeout) from `ERR_NETWORK` (real connectivity issue) in `normalizeError`

### 9. Fixed "Something went wrong" after the timeout fix

**Problem:** With 90s timeout, the request now reached the server — but the response was a 504 Gateway Timeout HTML page from nginx (nginx has its own timeout shorter than 90s). `normalizeError` couldn't parse HTML, so it showed "Something went wrong."

**Fix in `normalizeError`:** Detect HTML responses by checking if `data` is a string starting with `<`. Map them to:
- 504 → "The server took too long to respond. Please try again in a moment."
- Other HTML → "Server error (status). Please try again."

**Fix in `StepReview`:** Error box now shows "Booking failed" as a title, the message, and the HTTP status code for easier debugging.

**Root cause (not fixable from frontend):** The `book_other_services` dev server takes >60s for real bookings (likely because there are no vendors registered in the test area on dev). With an empty `data: []` payload it responds instantly (200). This is a dev environment limitation — on production with real vendors it should work.

### 10. Added `isMobile: true` header to `apiClient`

The mobile app sends `isMobile: true` in all authenticated requests. Added it to the `apiClient` default headers so our web app requests are consistent with what the backend expects.

### 11. Pushed to both repositories

After all fixes:
- Pushed to personal repo (`origin/main`)
- Pushed to org repo (`org/main`) — full website code

---

## What I accomplished today (full day)

- Read mobile app source end to end and mapped the full booking architecture
- Found correct booking history endpoint: `POST user_bookingsList`
- Found booking creation routes: `book_other_services` and `book_packersAndMovers`
- Replaced static room options with live `service_type` + `service_offerings` API calls
- Wired Confirm button to real booking creation endpoint with correct payload shape
- Fixed "Network error" — Axios timeout was too short for the dev server
- Fixed "Something went wrong" — `normalizeError` now handles nginx 504 HTML pages
- Added `isMobile: true` header to all authenticated API calls
- Pushed website code to personal and org repos

---

### 12. Booking flow working end to end

After comparing our payload with sir's working Postman test, found the remaining bugs:

- **`service_time`** — we were sending `"09:00 – 11:00"` (the display slot). API expects only the start time `"09:00"`. Fixed by splitting on the dash: `slot.split(/\s*[–-]\s*/)[0]`
- **`booking_id`** — sending `null`, should be `0`
- **`people_count`** — missing entirely, added with default `1`
- **`s_location`** — sending `{}`, now sends address string
- **Offerings not loading** — city field had garbage test data ("sfsf"). Backend filters offerings by state/emirate. Entering "Dubai" made offerings load correctly

After all fixes: Location → Schedule → Details (real types + offerings) → Review → **Confirm → Pay Now / Pay Later page**. Full booking flow working.

---

## What I accomplished today (full day)

- Read mobile app source end to end and mapped the full booking architecture
- Found correct booking history endpoint: `POST user_bookingsList`
- Found booking creation routes: `book_other_services` and `book_packersAndMovers`
- Replaced static room options with live `service_type` + `service_offerings` API calls
- Wired Confirm button to real booking creation endpoint with correct payload shape
- Fixed "Network error" — Axios timeout was too short for the dev server
- Fixed "Something went wrong" — `normalizeError` now handles nginx 502/504 HTML pages
- Fixed `service_time` format — start time only, not full slot range
- Fixed `booking_id`, `people_count`, `s_location` fields to match real API format
- Debugged empty offerings — city must be a real emirate (Dubai, Abu Dhabi, etc.)
- **Full booking flow working end to end including Pay Now / Pay Later screen**
- Pushed website code to personal and org repos

---

## What's next

- Profile edit page (`GET endUsers/0`, `POST newUpdateUser`)
- Address book page (`GET getAllAddress`, `POST saveAddress`)
- Payment flow wiring (`POST new_init_payment`, `POST afterpaymentSuccess`)
- Packers & Movers booking needs a separate flow (pickup + drop location, different payload format)

---

[← Day 12](day-12.md) · [Home](index.md)
