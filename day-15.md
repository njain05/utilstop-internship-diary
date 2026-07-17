# 09 July 2026 — Day 15: Home Page Categories, Step 3 Overhaul, Security Fix

## What was the task?

After getting the payment flow working yesterday, the focus shifted to user experience — the home page was a flat grid with no structure, Step 3 (service details) was showing raw data with no descriptions, and hardcoded API keys were sitting in `Constants.js`. All three needed fixing.

---

## What I did

### 1. Merged latest from org/developer

Started the day by fetching and merging `org/developer` into `nishtha-dev`. No conflicts — clean merge.

---

### 2. Navigation: Services dropdown → direct booking

**Problem:** The navbar "Services" dropdown linked to `/services/:serviceType` which went to `ServiceDetailPage`. Users had to read an info page and then click "Book Now" — an unnecessary extra step.

**Fix:** Changed the navbar dropdown links to point directly to the booking page. `ServiceDetailPage` now redirects immediately to the booking flow. Removed the duplicate "Home" link in the navbar too.

---

### 3. BookingFlowPage: service header and collapsible info panel

The booking flow had no visual context about which service you were booking. Added:

- **Service icon + region chip** — reads from URL params, shows the service icon and whether you're booking in UAE or India
- **"About this service" collapsible panel** — calls the `service_tier` API to get a description. Collapsed by default, opens on click.

---

### 4. Step 3 — offering descriptions

**Before:** Offering cards showed just the name. No context about what the offering includes.

**After:**
- Each offering card shows `short_description` below the name
- When selected, `description_list` appears as bullet points (what's included)
- If the offering has a `description_image`, it shows below the bullet list

All these fields were already being fetched — just never rendered.

---

### 5. Step 3 — service type UI overhaul

**Before:** Service types were a plain list.

**After:**
- Service types displayed as a **grid of cards** with icon, name, and `short_description`
- When a type is selected, a description panel **automatically expands below the grid**
- Offering `description_list` is **always visible** — no extra click required
- Offering selection shown with a **radio-style checkmark ring** — clearer than a colour change

---

### 6. Security fix — hardcoded API keys removed

During the day, noticed `Constants.js` had Google Maps API keys and the Stripe publishable key committed as plain strings.

**Fixed:**
- Moved `VITE_GOOGLE_MAPS_API_KEY`, `VITE_GOOGLE_PLACES_API_KEY`, and `VITE_STRIPE_PUBLISHABLE_KEY` to `.env` (which is `.gitignore`d)
- `Constants.js` now reads from `import.meta.env.VITE_*`
- Added `.env.example` with placeholders so other developers know what to set

---

### 7. Fix — `import.meta.env` crash in Node.js context

**Bug:** After moving keys to `import.meta.env`, the app crashed at startup.

**Root cause:** `vite.config.js` imports `Constants.js` at Vite startup — that runs in Node.js where `import.meta.env` doesn't exist.

**Fix:** Changed `import.meta.env.VITE_GOOGLE_MAPS_API_KEY` to `import.meta.env?.VITE_GOOGLE_MAPS_API_KEY`. Optional chaining returns `undefined` in Node.js instead of throwing.

---

### 8. CSP updated

Updated the Content Security Policy in `index.html` to allow:
- `maps.googleapis.com` and `maps.gstatic.com` — for Google Maps
- `api.stripe.com` and `js.stripe.com` — for Stripe Elements
- `fonts.googleapis.com` and `fonts.gstatic.com` — for Google Fonts

---

### 9. Fixed the Vite proxy double-path bug

**Bug:** API calls were hitting `/api/api/v1/...` instead of `/api/v1/...`.

**Fix:** Changed the proxy `target` to the base backend URL and updated the rewrite to pass `/api/v1/...` through intact.

---

### 10. Home page — service categories

**Before:** Home page showed a flat grid of all services.

**After:** Services grouped into four sections matching the mobile app:
- **Express** — fast-turnaround services
- **Popular** — top-booked services
- **Indoor** — services done inside the home
- **Outdoor** — services done outside

Each service card links directly to the booking page.

---

## Problems faced

- **`import.meta.env` crash** — `vite.config.js` runs in Node.js at startup, not in Vite's browser context. Fixed with optional chaining.
- **Vite proxy double-path** — fixed by correcting the target URL and rewrite rule.

---

## What I accomplished

- Navbar Services dropdown now goes straight to booking (bypasses detail page)
- BookingFlowPage: service icon + region header, collapsible "About this service" panel
- BookingSuccessPage: download app banner with App Store and Google Play links
- Fixed Vite proxy double-path bug
- CSP updated to allow Google Maps, Stripe, and Google Fonts
- Step 3: offering descriptions, bullet points, description_image
- Step 3: service type grid cards with auto-expanding description, radio-style selection
- Removed hardcoded API keys from Constants.js — moved to `.env`
- Fixed `import.meta.env` crash in Node.js context
- Home page: Express, Popular, Indoor, Outdoor service categories

---

## What's next

- Profile edit page
- Address book page
- Packers & Movers flow testing with real addresses
- Payment flow end-to-end test with Stripe test mode

---

[← Day 14](day-14.md) · [Home](index.md) · [Day 16 →](day-16.md)
