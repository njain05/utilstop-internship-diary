# 13 July 2026 — Day 17: Service Detail Pages, ScrollReveal, P&M Drawer Fix, Distance Bug

## What was the task?

Pick up from where Day 16 left off — add missing service detail pages, fix inconsistent animation behaviour across pages, sort out the "Additional Information" modal in Packers & Movers, and investigate the persistent "1 KM" distance badge bug.

---

## What I did

### 1. Service detail pages — Subscription Plans & Water Tank Cleaning

Added two new entries to `SERVICE_DATA` in `ServiceDetailPage.jsx`:

**`service_package_deals` — Subscription Plans**
Read the mobile app source (`subscribe/Subscription.js`) to understand what this is: recurring service bookings where the user picks a service and days of the week, and it auto-books on repeat at a discounted rate. Wrote the detail page with this explanation since the service name alone isn't self-explanatory.

**`service_watertank` — Water Tank Cleaning**
UAE municipality-approved tank cleaning service. Standard detail page with a why-choose section, FAQ, and book CTA.

Also added `heroBg` support in the hero section — services without a suitable photo can now use a CSS gradient string instead of an image path.

---

### 2. Landscaping hero image

Changed the landscaping service hero from `handyman-hero.png` to a gradient, then reverted back to `handyman-hero.png` after review. No outdoor/garden images exist in the project assets, so the handyman image is still the closest match.

---

### 3. Scroll animations — shared ScrollReveal component

**Problem:** Scroll animations only existed on `HomePage.jsx` as local components. Other pages (Service List, Partner, Career, Service Detail) had no animations. Also, animations only triggered once — scrolling back up didn't re-trigger them.

**Fix:** Extracted `ScrollReveal` and `StaggerReveal` into `src/components/ui/ScrollReveal.jsx`. Key change: `useInView` now uses `{ once: false }` so animations trigger in both directions.

Updated imports in:
- `HomePage.jsx` — removed local definitions
- `ServiceDetailPage.jsx` — wrapped all content sections
- `ServiceListPage.jsx` — wrapped header and service grid
- `PartnerPage.jsx` — wrapped header, benefits, and form
- `CareerPage.jsx` — wrapped header, why cards, positions, and apply form

---

### 4. Stagger crash in HomePage

After extracting to the shared file, `HomePage.jsx` crashed with `stagger is not defined`. The hero `motion.div` used `variants={stagger}` directly, but `stagger` was no longer defined locally.

**Fix:** Exported `stagger` and `fadeUp` from `ScrollReveal.jsx` and added them to the import in `HomePage.jsx`.

---

### 5. Additional Information — center modal → right-side drawer

**Problem:** The "Additional Information" button in `StepAddItems.jsx` (Packers & Movers step 2) opened a center-screen modal, inconsistent with the `OfferingViewDrawer` used everywhere else (slides from the right).

**Fix:** Replaced the center modal with an `AnimatePresence` drawer:
- Backdrop fades in/out
- Drawer slides in from the right with spring animation (`stiffness: 340, damping: 34`)
- Package name in header, bullet points staggered on entry, "Got it" footer button

---

### 6. Distance "1 KM" bug investigation

The distance badge between Dubai and Sharjah in Packers & Movers step 1 showed "1 KM" instead of ~40 KM.

**Root causes found:**

1. **Distance Matrix API not enabled** — GCP key doesn't have Distance Matrix API activated. Callback fires with a non-OK status, hitting `.catch(() => setDistanceKm(1))`.

2. **Origins/destinations swapped** — `getDistanceKm` was passing destination as origin and vice versa.

3. **Synchronous constructor throw** — `new DistanceMatrixService()` throws synchronously when the API isn't enabled, bypassing the Promise executor fallback.

**Fixes applied to `googleMaps.js`:**
- Added `haversineKm()` for straight-line distance as a fallback
- Wrapped service instantiation in try-catch inside the Promise executor
- Corrected swapped origins/destinations

**Coordinates confirmed from API URL:** pickup `25.0779, 55.13` (Dubai), drop `25.3309, 55.42` (Sharjah) — Haversine ≈ 40 KM. Correct.

---

### 7. Workflow issue — pull gap identified and fixed

Working in a git worktree and pushing to GitHub created a gap: the dev server runs from the main checkout which needed a separate pull to sync. Fixes were pushed to GitHub but the dev server still showed old code.

**Fix:** After every push, the main checkout is now auto-pulled.

---

## Problems faced

- **Stagger crash after ScrollReveal extraction** — `stagger` was used as a local variable that no longer existed after the refactor. Fixed by exporting it from the shared file.
- **Distance Matrix API disabled** — Can't fix from frontend. Used Haversine as fallback and flagged to Mani sir to enable the API on GCP.
- **Dev server showing stale code** — Worktree push didn't auto-sync the main checkout. Fixed workflow going forward.

---

## What I accomplished

- Added Subscription Plans and Water Tank Cleaning service detail pages
- Added gradient hero (`heroBg`) support as alternative to image heroes
- Extracted `ScrollReveal` / `StaggerReveal` into shared component with bidirectional animation
- Added scroll animations across Service List, Partner, Career, and Service Detail pages
- Fixed stagger crash in HomePage
- Converted Additional Information popup to right-side spring drawer
- Fixed Distance Matrix — Haversine fallback, try-catch, corrected swapped origins
- Fixed the pull workflow gap

---

## What's next

- Verify distance fix is working (should show ~40 KM for Dubai → Sharjah after hard refresh)
- Ask Mani sir to enable Distance Matrix API on GCP key for accurate road distances
- Profile edit, Address Book, Notifications, Wallet pages

---

[← Day 16](day-16.md) · [Home](index.md) · [Day 18 →](day-18.md)
