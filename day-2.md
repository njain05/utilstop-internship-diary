# 23 June 2026 — Day 2: Figma Deep-Dive

## What was the task?
Study the Figma mobile app design thoroughly. Map every screen to a route. Understand the user journey from landing to booking completion.

---

## What did we do?

- Spent the full day inside the Figma file
- Inventoried every screen in the mobile app:
  - Splash / onboarding
  - Home screen (service grid)
  - Service detail
  - Booking flow (multi-step — date, time, address, requirements)
  - "Offer Your Price" / bid submission screen
  - Bid status — waiting for vendor to accept
  - Bid accept confirmation
  - Payment — Pay Now vs Pay Later choice
  - Live tracking screen
  - Booking history
  - Profile / account settings
  - Login, Register, Forgot Password
- Noted that the mobile app uses a **bottom tab bar** — on web this will become a **top navbar**
- Noted the design system: Inter font, `#C81849` primary color, rounded cards, clean white surfaces
- Mapped every screen to a web URL route (e.g. `/services/:slug/book`, `/booking/bids/:id`, `/tracking/:id`)

---

## Problems faced

- The Figma file has mobile frames (375px wide) — every layout needs to be rethought for 1440px desktop
- Some screens in Figma are incomplete or have placeholder text — I'll have to fill in the gaps logically
- The bidding flow is the most complex: user submits a price → vendors respond → user accepts one bid → proceeds to payment. This is a 3-4 screen sub-flow I need to design myself for web

---

## What I accomplished

- Complete screen inventory (21 unique screens identified)
- Full route map drafted on paper
- Design decisions made: top navbar replaces bottom tabs, 2-3 column grids for desktop, same brand colors
- Understood the most security-critical part: the bid amount is user-typed text that must be validated and sanitized

---

[← Day 1](day-1.md) · [Home](index.md) · [Day 3 →](day-3.md)
