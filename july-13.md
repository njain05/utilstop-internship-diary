# Developer Diary — 13 July 2026

[← 10 July](july-10.md) · [Home](index.md)

---

## Overview

Long debugging session focused on Packers & Movers booking flow, scroll animations, and service detail pages. Added two new service detail pages, extracted scroll animations into a shared component, fixed a distance calculation bug, converted the Additional Information modal into a right-side drawer, and investigated the persistent "1 KM" distance badge.

---

## 1. Service detail pages — Subscription Plans & Water Tank

Added two new entries to `SERVICE_DATA` in `ServiceDetailPage.jsx`:

**`service_package_deals` — Subscription Plans**
Looked into the mobile app source (`subscribe/Subscription.js`) to understand what this actually is: recurring service bookings where the user picks a service and days of the week, and it auto-books on repeat at a discounted rate. Wrote the detail page with this explanation since it wasn't obvious from the service name alone.

**`service_watertank` — Water Tank Cleaning**
UAE municipality-approved tank cleaning service. Standard detail page with why-choose section, FAQ, and book CTA.

Also added `heroBg` support in the hero section — services without a suitable photo can now use a CSS gradient string instead of an image path.

---

## 2. Landscaping hero image

Changed the landscaping service hero from `handyman-hero.png` to a gradient, then reverted it back to `handyman-hero.png` after review. No outdoor/garden images exist in the project assets, so the handyman image is still the closest match.

---

## 3. Scroll animations — shared ScrollReveal component

**Problem:** Scroll animations only existed on `HomePage.jsx` as local components. Other pages (Service List, Partner, Career, Service Detail) had no animations at all. Also, animations only triggered once on scroll-down — scrolling back up didn't re-trigger them.

**Fix:** Extracted `ScrollReveal` and `StaggerReveal` into a new shared file `src/components/ui/ScrollReveal.jsx`. Key change: `useInView` now uses `{ once: false }` so elements animate both when entering and leaving the viewport.

Updated imports in:
- `HomePage.jsx` — removed local definitions, imported from shared file
- `ServiceDetailPage.jsx` — wrapped all content sections
- `ServiceListPage.jsx` — wrapped header and service grid
- `PartnerPage.jsx` — wrapped header, benefits, and form
- `CareerPage.jsx` — wrapped header, why cards, positions, and apply form

---

## 4. Stagger crash in HomePage

After extracting to the shared file, `HomePage.jsx` crashed with `stagger is not defined`. The hero `motion.div` used `variants={stagger}` directly, but `stagger` was no longer defined locally. Fixed by exporting `stagger` and `fadeUp` from `ScrollReveal.jsx` and adding them to the import in `HomePage.jsx`.

---

## 5. Additional Information — center modal → right-side drawer

**Problem:** The "Additional Information" button in `StepAddItems.jsx` (Packers & Movers step 2) opened a center-screen modal, unlike the `OfferingViewDrawer` used elsewhere which slides in from the right.

**Fix:** Replaced the center modal with an `AnimatePresence` drawer:
- Backdrop fades in/out
- Drawer slides in from the right with spring animation (`stiffness: 340, damping: 34`)
- Package name in header, bullet points staggered on entry, "Got it" footer button
- Matches the design pattern used in the rest of the booking flow

---

## 6. Distance "1 KM" bug investigation

The distance badge between Dubai and Sharjah in Packers & Movers step 1 showed "1 KM" instead of the actual ~40 KM.

**Root causes found:**

1. **Distance Matrix API not enabled** — The GCP key doesn't have Distance Matrix API activated. The callback fires with a non-OK status, hitting `.catch(() => setDistanceKm(1))`.

2. **Origins/destinations swapped** — The original `getDistanceKm` passed `destination` as origin and vice versa. Doesn't affect the distance value but was incorrect.

3. **Synchronous constructor throw** — `new DistanceMatrixService()` throws synchronously when the API isn't enabled, escaping the Promise executor and bypassing the callback fallback logic.

**Fixes applied to `googleMaps.js`:**
- Added `haversineKm()` for straight-line distance as a fallback
- Wrapped the service instantiation in try-catch inside the Promise executor
- Corrected swapped origins/destinations
- Added debug logging to trace coordinate values

**Coordinates confirmed from API request URL:** pickup `25.0779, 55.13` (Dubai), drop `25.3309, 55.42` (Sharjah) — Haversine of these ≈ 40 KM, which is correct.

**Status:** Fix is pushed. The "1 KM" was caused by the browser running cached old code — the local checkout wasn't being synced after each push from the worktree.

---

## 7. Workflow issue — pull gap identified and fixed

Working in a git worktree (`.claude/worktrees/fix-service-slug`) and pushing to GitHub created a gap: the dev server runs from the main checkout (`/Users/cheshtha/nishtha_jain/utilstop_website/`) which needed a separate pull to sync. This caused repeated confusion where fixes were pushed to GitHub but the dev server still showed old code.

**Fix:** After every push, the main checkout is now auto-pulled so the dev server always has the latest code without a manual pull step.

---

## What I accomplished today

- Added Subscription Plans and Water Tank Cleaning service detail pages
- Added gradient hero (`heroBg`) support as alternative to image heroes
- Extracted `ScrollReveal` / `StaggerReveal` into shared component with bidirectional animation
- Added scroll animations across Service List, Partner, Career, and Service Detail pages
- Fixed stagger crash in HomePage
- Converted Additional Information popup to right-side spring drawer
- Fixed Distance Matrix issues — Haversine fallback, try-catch, swapped origins corrected
- Pushed all changes to both personal fork (`origin`) and org repo
- Fixed the pull workflow gap — main checkout now auto-synced after each push

---

## What's next

- Verify distance fix is working (should show ~40 KM for Dubai → Sharjah after hard refresh)
- Ask Mani sir to enable Distance Matrix API on GCP key for accurate road distances
- Profile edit, Address Book, Notifications, Wallet, Coupons, Subscriptions pages

---

[← 10 July](july-10.md) · [Home](index.md)
