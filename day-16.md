# Developer Diary — Day 16 (10 July 2026)

[← Day 15](day-15.md) · [Home](index.md)

---

## Overview

Focused on the Packers & Movers / Item Moving booking flow, cross-repo housekeeping, and two PRs. Fixed the location picker (Enter key, country restriction, state matching), replaced the tracking page with an app download prompt, updated the Google Maps key, and identified two backend blockers (Distance Matrix API not enabled, `/service_offerings` returning 500). Ended the day with a clean PR raised to the org `developer` branch.

---

## 1. Cross-repo housekeeping — diary and merges

Started by realising the `diary/` folder had been deleted from `nishtha-dev` in an earlier cleanup. Before merging PR #1 into `main` on the personal repo, restored the diary from `origin/main` and committed it back to `nishtha-dev` so the merge wouldn't delete it from `main`.

Merged PR #1 (`nishtha-dev` → `main`) on `njain05/utilstop-website` — 56 files, 2193 additions, 1578 deletions. Clean merge, no conflicts.

Also pulled `org/developer` into `nishtha-dev`. One conflict in `Constants.js` — both branches had changed the Google API key lines, with our branch having the optional chaining fix (`?.`) and `org/developer` without it. Kept our version since it prevents a Vite startup crash in Node.js context.

---

## 2. Country selection — already working

User reported not seeing the country selection modal on startup. Root cause: `localStorage` already had `hasSelectedCountry: true` from a previous session, so the modal correctly didn't show. Not a bug — expected behaviour. To re-test, clear `utilstop-location` from Application → Local Storage in DevTools.

---

## 3. Tracking page — replaced with app download prompt

The tracking page was showing a timeline UI that polled the backend for live status. Replaced it entirely with an app download page:

- Clear message that live tracking is **app-only**
- Gradient promo card: **"10% off your next booking"** with code `APPFIRST`
- Four app benefits listed (live map, push notifications, vendor chat, loyalty rewards)
- App Store and Google Play download buttons using the correct store URLs

The old tracking API hook and timeline component are no longer used on this page.

---

## 4. Google Maps key updated

Received a new Google Maps API key from Mani sir (`AIzaSyD-ps1...`) to fix `ApiTargetBlockedMapError` on the local setup. Updated `VITE_GOOGLE_PLACES_KEY` in `.env` to the new key. The old key had restrictions that blocked `localhost`.

---

## 5. Location picker — Enter key support

**Problem:** The Google Maps `Autocomplete` component only fires `onPlaceChanged` when the user clicks a suggestion from the dropdown. Pressing Enter did nothing.

**Fix:** Added a `keydown` handler on the search input — when Enter is pressed, calls `window.google.maps.event.trigger(autocompleteRef.current, 'place_changed')` programmatically. Also added Enter-key navigation through the address detail form:
- Floor/Flat No. → moves focus to Building Name
- Building Name → moves focus to Landmark
- Landmark → submits the form

---

## 6. Location picker — country-aware autocomplete

**Problem:** `AUTOCOMPLETE_OPTIONS` had `componentRestrictions: { country: 'ae' }` hardcoded. Indian users searching for Indian addresses got no results.

**Fix:** Read the selected country from `useLocationStore` inside the modal. Autocomplete now restricts to `'ae'` for UAE and `'in'` for India. The state dropdown also switches between `UAEStates` and `IndiaStates` from the `getAllStates` API. The label changes from "Emirate" to "State" for Indian users.

---

## 7. Location picker — state matching fix

**Problem:** After selecting an address, the modal showed "Our services are currently available only in Dubai" even for valid Dubai addresses. The Emirate dropdown also showed nothing.

**Root causes (two separate issues):**

1. **Wrong API response accessor** — `getAllStates` returns `r.data` from Axios, so the response in the component is already the data body. The code was reading `res?.data?.UAEStates` which looked for `data.data.UAEStates` — an extra level of nesting. Fixed to `(res?.data ?? res)?.UAEStates`.

2. **Case-sensitive state matching** — `matchState` used `adr.includes(s)` which is case-sensitive. If the API returns `"dubai"` but Google's address has `"Dubai"`, no match. Fixed to lowercase both sides.

3. **State validation blocks when list is empty** — if `getAllStates` fails to load, `states` is `[]` and every address gets rejected. Fixed to skip the state check entirely when `states.length === 0`, letting the backend validate instead.

---

## 8. Packers & Movers / Item Moving — same mobile_redirect

Noticed that clicking "Packers & Movers" on the home page opened "Item Moving" in the booking flow. Both services share the same `mobile_redirect` value (`service_packers_movers`), so `useServiceBySlug`'s `data.find()` returns whichever comes first in the API response — which is Item Moving.

Considered fixing this by routing via `service.id` instead. Decided against it — `mobile_redirect` is used for deep linking from the mobile app and changing the URL structure could break that. The two services intentionally share the same booking flow (`PackersBookingFlow`), so the UX impact is minimal. Left as-is.

---

## 9. Step 2 blank screen — missing local state guard

**Problem:** `movingDetails` (pickup/drop addresses, date/time) is local component state in `PackersBookingFlow`. If the Zustand store has `currentStep: 2` persisted but the component remounts (page refresh, navigation), `movingDetails` is null. The condition `currentStep === 2 && movingDetails &&` rendered nothing — not even a back button. User was stuck.

**Fix:** Changed the condition to always render something at step 2:
- If `movingDetails` exists → render `StepAddItems` normally
- If `movingDetails` is null → render a fallback message with a "Go back" link to step 1

---

## 10. Step 2 backend 500 — two issues identified

The `/service_offerings/{serviceId}` endpoint returns `500` for the Item Moving service. Console showed two related errors:

1. **`LegacyApiNotActivatedMapError`** — Distance Matrix API is not enabled on the GCP key. `getDistanceKm()` fails silently and falls back to `1`. The backend may be rejecting a distance of `1` as invalid.

2. **500 from `/service_offerings`** — possibly no distance-based offerings configured for Item Moving in the backend admin panel.

Sent a message to Mani sir to:
- Enable Distance Matrix API on the key
- Confirm Item Moving has offerings configured
- Enable Places API (New) to clear the Autocomplete deprecation warning

These are backend/GCP issues — nothing to fix on the frontend until they're resolved.

---

## 11. PR raised to org/developer

Raised PR #4 on `UtilStop-L-L-C/utilstop_website` — `nishtha-dev` → `developer`. No conflicts. Files included:

- `src/api/services.api.js` — `getPeopleAlsoBook` API call
- `src/components/booking/packersMovers/LocationPickerModal.jsx` — all location picker fixes
- `src/components/booking/packersMovers/StepAddItems.jsx` — better error message
- `src/constants/Constants.js` — optional chaining fix
- `src/pages/booking/BookingFlowPage.jsx` — offerings modal, bundle deals sidebar, step guard
- `src/pages/public/HomePage.jsx` — service categories
- `src/pages/tracking/TrackingPage.jsx` — app download prompt

Diary folder was removed from `org/nishtha-dev` before raising the PR (using a temp branch so the personal repo's branch was not affected).

---

## What I accomplished today

- Merged PR #1 into `main` on personal repo (clean, no conflicts)
- Merged `org/developer` into `nishtha-dev`, resolved one conflict in `Constants.js`
- Replaced tracking page with app download prompt + 10% off incentive
- Updated Google Maps key in `.env`
- Location picker: Enter key support on search field and form fields
- Location picker: country-aware autocomplete (UAE/India)
- Location picker: fixed `getAllStates` response parsing, case-insensitive state matching, skip validation when states list is empty
- Fixed blank screen on booking step 2 when local state is missing
- Identified two backend blockers for Item Moving step 2 — flagged to Mani sir
- Raised PR #4 to `org/developer`, diary excluded from org repo

---

## What's next

- Wait for Mani sir to enable Distance Matrix API and confirm Item Moving offerings in backend
- Payment flow debugging — `bookingId` is likely `'new'` (fallback), need to confirm actual API response shape
- Profile edit and Address Book pages
- Test full Packers & Movers flow end-to-end once backend issues are resolved

---

[← Day 15](day-15.md) · [Home](index.md)
