# Developer Diary — 9 July 2026

[← 08 July](july-08.md) · [Home](index.md)

---

## Overview

Productive day. Focused on the home page, navigation UX, and the booking Step 3 experience. Also did a security pass — found hardcoded API keys in `Constants.js` and moved them to `.env`. Several bugs fixed along the way. By end of day the home page looks much closer to the mobile app, and Step 3 (service types + offerings) is far more informative and usable.

---

## 1. Merged latest from org/developer

Started the day by fetching and merging `org/developer` into `nishtha-dev`. No conflicts — the org branch was behind our branch, so it was a fast-forward-style merge. Nothing new from the org side today.

---

## 2. Navigation: Services dropdown → direct booking

**Problem:** The navbar "Services" dropdown linked to `/services/:serviceType`, which went to `ServiceDetailPage`. That page had an info panel and a "Book Now" button. Users had to read the page and then click through to start booking — an unnecessary extra step.

**Fix:** Changed the navbar dropdown links to point directly to `/book?service=<serviceType>`. The `ServiceDetailPage` now just redirects immediately to the booking flow. No more dead-end info page in the way.

**Also fixed:** The "Home" link appeared twice in the navbar. Removed the duplicate.

---

## 3. BookingFlowPage: service header and collapsible info panel

The booking flow had no visual context about which service you were booking. Added two things to the header:

- **Service icon + region chip** — reads from the URL params (`service` and `country` query strings), shows the service icon and whether you're booking in UAE or India. Small UI detail but makes the page feel intentional.
- **"About this service" collapsible panel** — calls the `service_tier` API endpoint (same one used for offerings) to get a description of the service. The panel is collapsed by default and opens on click. Content shows what the service covers, typical use cases, etc.

---

## 4. BookingSuccessPage: download app banner

After a successful booking, users see a confirmation screen. Added a banner at the bottom encouraging them to download the mobile app — App Store and Google Play buttons with the correct store links. This matches the mobile app's post-booking behaviour.

---

## 5. Fixed the Vite proxy double-path bug

**Bug:** API calls were hitting `/api/api/v1/...` instead of `/api/v1/...`.

**Root cause:** The Vite proxy `rewrite` rule was stripping `/api` from the path, then forwarding to the target which was `http://backend/api/v1`. So the `/api` was being added back by the target. The path ended up doubled.

**Fix:** Changed the proxy `target` to `http://backend` (without `/api/v1`) and let the rewrite pass through `/api/v1/...` intact. One clear path, no doubling.

---

## 6. CSP updated

The Content Security Policy meta tag in `index.html` was blocking some legitimate requests. Updated to allow:

- `maps.googleapis.com` and `maps.gstatic.com` — for Google Maps
- `api.stripe.com` and `js.stripe.com` — for Stripe Elements
- `fonts.googleapis.com` and `fonts.gstatic.com` — for Google Fonts used by Stripe

All sources are explicit — no wildcards added.

---

## 7. Step 3: Offering descriptions

**Before:** Offering cards showed just the name. No context about what the offering actually includes.

**After:**
- Each offering card now shows `short_description` below the name — a one-liner about what it covers
- When an offering is selected, `description_list` appears as bullet points (what's included in the offering)
- If the offering has a `description_image`, it shows below the bullet list — useful for Home Cleaning tiers that have visual comparisons

These fields all come from the `service_offerings` API response — they were already being fetched, just never rendered.

---

## 8. Step 3: Service type UI overhaul

**Before:** Service types were a list — you clicked one and the offerings below changed. Not obvious what each type meant without clicking around.

**After:**
- Service types displayed as a **grid of cards**, each showing: icon, name, and `short_description`
- When a type is selected, a description panel **automatically expands below the grid** (no extra click required) — shows more detail from `description` field
- Offering `description_list` is **always visible** — not hidden behind a click. You can compare offerings without interacting first
- Offering selection is shown with a **radio-style checkmark** ring — clearer visual state than a background colour change

---

## 9. Security fix: hardcoded API keys removed

During the day, noticed `Constants.js` had Google Maps API keys and the Stripe publishable key committed as plain strings. This was from the previous day's integration work.

**Fixed:**
- Moved `VITE_GOOGLE_MAPS_API_KEY`, `VITE_GOOGLE_PLACES_API_KEY`, and `VITE_STRIPE_PUBLISHABLE_KEY` to `.env` (which is `.gitignore`d)
- `Constants.js` now reads from `import.meta.env.VITE_*` — if the key isn't set, the value is `undefined` rather than a hardcoded string

Added `.env.example` with placeholders for all three keys so other developers know what to set.

---

## 10. Fix: `import.meta.env` crash in Node.js context

**Bug:** After moving keys to `import.meta.env`, the app crashed at startup with `Cannot read properties of undefined (reading 'VITE_GOOGLE_MAPS_API_KEY')`.

**Root cause:** `vite.config.js` imports `Constants.js` to read the proxy target. That import runs in Node.js at Vite startup — `import.meta.env` doesn't exist in Node.js, so it's `undefined`.

**Fix:** Changed `import.meta.env.VITE_GOOGLE_MAPS_API_KEY` to `import.meta.env?.VITE_GOOGLE_MAPS_API_KEY` in `Constants.js`. Optional chaining means the expression safely returns `undefined` in Node.js context instead of throwing.

---

## 11. Home page: service categories

**Before:** The home page showed a flat grid of all services.

**After:** Services are now grouped into four sections matching the mobile app's `home.js` category logic:
- **Express** — fast-turnaround services (Pest Control, Disinfection)
- **Popular** — top-booked services (Home Cleaning, AC Service)
- **Indoor** — services done inside the home (Appliance Repair, Painting)
- **Outdoor** — services done outside (Landscaping, Pool Cleaning)

Each service card now links directly to `/book?service=<type>` rather than going through the detail page.

The category mapping was ported directly from the mobile app's `home.js` constants — same service IDs, same groupings.

---

## What I accomplished today

- Navbar Services dropdown now bypasses the detail page and goes straight to booking
- ServiceDetailPage cleaned up — dead code removed, now just redirects
- BookingFlowPage: service icon + region header, collapsible "About this service" panel
- BookingSuccessPage: download app banner with App Store and Google Play links
- Fixed Vite proxy double-path bug (`/api/api/v1` → `/api/v1`)
- CSP updated to allow Google Maps, Stripe, and Google Fonts
- Step 3: offering short descriptions, description_list bullet points, description_image
- Step 3: service type grid cards with auto-expanding description panel
- Step 3: offering description always visible, radio-style selection state
- Removed hardcoded API keys from Constants.js — moved to .env
- Fixed `import.meta.env` crash in Node.js context (optional chaining)
- Home page: Express, Popular, Indoor, Outdoor service categories
- ServiceCard links directly to booking (no extra click)

---

## What's next

- Profile edit page (`GET endUsers/0`, `POST newUpdateUser`)
- Address book page (`GET getAllAddress`, `POST saveAddress`)
- Packers & Movers flow testing with real addresses (needs Google Maps key)
- Payment flow end-to-end test with Stripe test mode

---

[← 08 July](july-08.md) · [Home](index.md)
