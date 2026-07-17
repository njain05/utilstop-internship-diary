# 10 July 2026 — Day 16: Location Picker Fixes, Tracking Page, PR to org/developer

## What was the task?

The Packers & Movers location picker had several UX bugs — Enter key didn't work, country selection was hardcoded to UAE, and state matching was rejecting valid addresses. The tracking page was also showing a fake timeline. Needed to fix all of this and raise a clean PR to the org developer branch.

---

## What I did

### 1. Cross-repo housekeeping — diary and merges

Started by realising the `diary/` folder had been deleted from `nishtha-dev` in an earlier cleanup. Restored the diary from `origin/main` and committed it back to `nishtha-dev` before merging.

Merged PR #1 (`nishtha-dev` → `main`) on `njain05/utilstop-website` — 56 files, 2193 additions, 1578 deletions. Clean merge, no conflicts.

Also pulled `org/developer` into `nishtha-dev`. One conflict in `Constants.js` — both branches had changed the Google API key lines. Kept our version (with the `?.` optional chaining fix) since it prevents a Vite startup crash.

---

### 2. Country selection — confirmed working

User reported not seeing the country selection modal on startup. Root cause: `localStorage` already had `hasSelectedCountry: true` from a previous session — modal correctly didn't show. Not a bug. To re-test: clear `utilstop-location` from Application → Local Storage in DevTools.

---

### 3. Tracking page — replaced with app download prompt

The tracking page was showing a timeline UI polling the backend for live status. Replaced it with an app download page:

- Clear message that live tracking is **app-only**
- Gradient promo card: **"10% off your next booking"** with code `APPFIRST`
- Four app benefits listed (live map, push notifications, vendor chat, loyalty rewards)
- App Store and Google Play download buttons

---

### 4. Google Maps key updated

Received a new Google Maps API key from Mani sir to fix `ApiTargetBlockedMapError` on local. Updated `VITE_GOOGLE_PLACES_KEY` in `.env`.

---

### 5. Location picker — Enter key support

**Problem:** The Google Maps `Autocomplete` component only fires `onPlaceChanged` when the user clicks a suggestion. Pressing Enter did nothing.

**Fix:** Added a `keydown` handler — when Enter is pressed, calls `window.google.maps.event.trigger(autocompleteRef.current, 'place_changed')` programmatically. Also added Enter-key navigation through the address detail form fields.

---

### 6. Location picker — country-aware autocomplete

**Problem:** `AUTOCOMPLETE_OPTIONS` had `componentRestrictions: { country: 'ae' }` hardcoded. Indian users got no results.

**Fix:** Read selected country from `useLocationStore`. Autocomplete now restricts to `'ae'` for UAE and `'in'` for India. State dropdown switches between `UAEStates` and `IndiaStates`. Label changes from "Emirate" to "State" for India.

---

### 7. Location picker — state matching fix

**Problem:** After selecting an address, the modal showed "Our services are currently available only in Dubai" even for valid Dubai addresses.

**Root causes:**

1. **Wrong API response accessor** — Code was reading `res?.data?.UAEStates` (i.e. `data.data.UAEStates` — one extra level). Fixed to `(res?.data ?? res)?.UAEStates`.

2. **Case-sensitive state matching** — `adr.includes(s)` is case-sensitive. `"dubai"` ≠ `"Dubai"`. Fixed to lowercase both sides.

3. **State validation blocks when list is empty** — if `getAllStates` fails, `states` is `[]` and every address gets rejected. Fixed to skip the check when `states.length === 0`.

---

### 8. Packers & Movers / Item Moving — same mobile_redirect

Noticed clicking "Packers & Movers" on the home page opened "Item Moving" in the booking flow. Both services share the same `mobile_redirect` value, so `data.find()` returns whichever comes first in the API response. Left as-is — the two services intentionally share the same booking flow, and changing the URL structure could break mobile app deep links.

---

### 9. Step 2 blank screen — missing local state guard

**Problem:** If the Zustand store has `currentStep: 2` persisted but the component remounts, `movingDetails` is null. The condition `currentStep === 2 && movingDetails &&` rendered nothing — not even a back button.

**Fix:** Always render something at step 2 — if `movingDetails` is null, show a fallback message with a "Go back" link to step 1.

---

### 10. Step 2 backend 500 — backend blockers identified

The `/service_offerings/{serviceId}` endpoint returns `500` for Item Moving:

1. **`LegacyApiNotActivatedMapError`** — Distance Matrix API not enabled on GCP key. `getDistanceKm()` falls back to `1`. Backend may reject a distance of `1`.
2. **500 from `/service_offerings`** — possibly no distance-based offerings configured for Item Moving in the backend admin.

Flagged to Mani sir — nothing to fix on the frontend until resolved.

---

### 11. PR raised to org/developer

Raised PR #4 on `UtilStop-L-L-C/utilstop_website` — `nishtha-dev` → `developer`. No conflicts. Diary folder removed from org repo before raising.

---

## Problems faced

- **State matching rejecting valid addresses** — two separate bugs (wrong response accessor + case sensitivity). Both fixed.
- **Item Moving step 2 500 error** — backend/GCP issue, not fixable from frontend. Flagged to Mani sir.

---

## What I accomplished

- Merged PR #1 into `main` on personal repo
- Merged `org/developer` into `nishtha-dev`, resolved `Constants.js` conflict
- Replaced tracking page with app download prompt + 10% off incentive
- Updated Google Maps key in `.env`
- Location picker: Enter key support on search and form fields
- Location picker: country-aware autocomplete (UAE/India)
- Location picker: fixed response parsing, case-insensitive matching, skip validation when states empty
- Fixed blank screen on booking step 2 when local state is missing
- Raised PR #4 to `org/developer`

---

## What's next

- Wait for Mani sir to enable Distance Matrix API and confirm Item Moving offerings in backend
- Profile edit and Address Book pages
- Test full Packers & Movers flow end-to-end once backend issues are resolved

---

[← Day 15](day-15.md) · [Home](index.md) · [Day 17 →](day-17.md)
