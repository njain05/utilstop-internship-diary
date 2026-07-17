# 15 July 2026 — Day 19: CountryToggle, OfferingViewDrawer, Edit Links on Review Step

## What was the task?

After the team demo on Day 18, a few UX friction points needed attention. The country selection modal was blocking the home page on every visit, offering descriptions were duplicated across components instead of being shared, the order summary card was missing labels, and the Review step had no way to jump back and fix individual fields.

---

## What I did

### 1. Synced nishtha-dev with org/developer

Pulled the latest from `org/developer` into `nishtha-dev`. The merge brought in Sharanya's cart and guest-mode changes from PR #11. Also removed the now-redundant `BookingConfirmPage.jsx` — its functionality had been absorbed into the main booking flow.

---

### 2. CountryToggle — replaced modal with inline pill

**Problem:** `CountrySelectModal` was a blocking modal that appeared on every visit before the user could see anything.

**Fix:** Built a new `CountryToggle` component (`src/components/ui/CountryToggle.jsx`) — a small `India | UAE` pill that switches the country inline without any blocking popup. Added it to:
- Desktop header (right cluster, before auth elements)
- Mobile menu (top section)
- Home page hero (replaced the "Browse Services" button)

Also set `hasSelectedCountry: true` as the default in `locationStore.js` so first-time visitors land straight on the home page. The toggle is always visible instead.

---

### 3. OfferingViewDrawer — extracted reusable component

The "What's Included" drawer that shows an offering's description image and detail list was previously defined inline inside `BookingFlowPage.jsx`. Extracted it into `src/components/booking/OfferingViewDrawer.jsx` so it can be reused.

Updated `StepAddItems.jsx` (Packers & Movers) to import and use `OfferingViewDrawer` instead of its own inline modal — UI is now consistent across all service types.

---

### 4. OrderSummaryCard — DATE and TIME labels fixed

**Problem:** The date and time row in the payment summary card showed bare values with no labels — inconsistent with every other row which had a clear uppercase label.

**Fix:** Added `DATE` and `TIME` label typography (`text-xs font-semibold text-neutral-400 uppercase tracking-wide`) above each value, matching the existing row style.

---

### 5. Review step — Edit links added

Each row on the Review step (Location, Date, Time, Type, Offerings, Pricing) now has an inline **Edit** link that jumps directly back to the relevant step:
- Location → Step 1
- Date / Time → Step 2
- Type / Offerings / Pricing → Step 3

The `ReviewRow` helper was updated to accept an optional `onEdit` prop. `StepReview` now receives an `onGoToStep` callback from `BookingFlowPage`.

---

## Problems faced

- No major blockers today. All changes were straightforward UI fixes.

---

## What I accomplished

- Pulled org/developer into nishtha-dev, removed redundant BookingConfirmPage
- Built `CountryToggle` inline pill — no more blocking country modal
- Extracted `OfferingViewDrawer` into a shared component used across all service types
- Fixed missing DATE/TIME labels in OrderSummaryCard
- Added per-row Edit links on the Review step
- Pushed all changes to org/nishtha-dev

---

## What's next

- Convert Packers & Movers to a 4-step flow matching all other services
- Address the crowded desktop navbar — explore sidebar or mega-menu approach
- Raise PR from nishtha-dev to org/developer

---

[← Day 18](day-18.md) · [Home](index.md) · [Day 20 →](day-20.md)
