# 26 June 2026 — Day 5: Architecture + Public Pages

## What was the task?
Build the invisible backbone of the app — state management, API abstraction, routing skeleton, and the security layer. Then immediately move into building all the public-facing pages so there is something real to see in the browser by end of day.

---

## Part 1 — Architecture

### Zustand Stores (`src/store/`)
- `authStore.js` — holds `accessToken`, `user`, `isAuthenticated`, `isRefreshing`. Token is in-memory only — never written to `localStorage` (security requirement)
- `bookingStore.js` — tracks the multi-step booking being built: selected service, date, time, address, requirements, payment method, current step
- `locationStore.js` — tracks whether the user is in UAE or India, and their selected city

### API Layer (`src/api/`)
- `client.js` — Axios instance with base URL from env, 15-second timeout, `withCredentials: true`
  - **Request interceptor**: reads the access token from Zustand and attaches it as `Authorization: Bearer <token>` to every outgoing request
  - **Response interceptor**: if a 401 comes back, automatically tries to refresh the token via `/auth/refresh`, then retries the original request. If refresh fails, logs the user out
- `auth.api.js`, `booking.api.js`, `bid.api.js`, `payment.api.js`, `services.api.js`, `tracking.api.js`, `vendor.api.js` — one file per domain
- `index.js` — reads `VITE_MOCK_API` env var and exports either real API modules or mock modules

### Routes (`src/routes/`)
- `routes.js` — all route path constants in one place (no magic strings)
- `PrivateRoute.jsx` — redirects to `/login` if not authenticated, saves the intended destination in location state
- `PublicOnlyRoute.jsx` — redirects to `/account` if already logged in
- `AppRouter.jsx` — skeleton with all route paths defined

### Security utilities
- `sanitize.js` — DOMPurify wrapper strips any HTML/script tags from user input
- `errorNormalizer.js` — normalizes API errors into a consistent `{ message, code, status }` shape
- Zod schemas for every domain: `auth.schema.js`, `bid.schema.js`, `booking.schema.js`, `address.schema.js`, `payment.schema.js`

---

## Part 2 — Public Pages + Reusable UI Components

### Reusable UI components (`src/components/ui/`)
Built the shared building blocks that every page will use:
- `Button.jsx` — variants: primary, secondary, ghost. Sizes: sm, md, lg. Has a `loading` prop (shows spinner, disables click)
- `Input.jsx` — label, error message, required indicator, forwarded ref for React Hook Form
- `Card.jsx` — white rounded card with border
- `Badge.jsx` — small colored pill labels
- `Spinner.jsx` — animated loading indicator
- `Skeleton.jsx` — placeholder shimmer for loading states

### Home Page (`src/pages/public/HomePage.jsx`)
6 sections, all built today:
1. **Hero** — full-width crimson background, headline, city search input, Browse Services CTA, stat chips (10,000+ bookings, 500+ verified pros)
2. **Services grid** — 2-col mobile / 3-col desktop, custom inline SVG icons per service category
3. **How It Works** — 3-step process with numbered cards
4. **Why UtilStop** — 6 feature cards (0% commission, bid pricing, verified pros, Pay Now/Later, UAE & India, Express)
5. **Promo banner** — "Get 40% off" section with two CTAs
6. **App download** — mock phone illustration, App Store and Google Play buttons

### Login Page (`src/pages/public/LoginPage.jsx`)
- React Hook Form + Zod validation
- Email and password fields with inline error messages
- Server error banner for wrong credentials
- Loading state on submit button
- Redirects to the originally intended page after login (saved in location state by `PrivateRoute`)

### Register Page (`src/pages/public/RegisterPage.jsx`)
- Name, email, phone, password, confirm password — all validated via Zod schema
- Same UX pattern as Login

### Forgot Password Page (`src/pages/public/ForgotPasswordPage.jsx`)
- Single email field
- Success state: "check your inbox" message shown after submission

---

## Problems faced

- The response interceptor has a subtle infinite loop risk: if `/auth/refresh` itself returns a 401, the interceptor fires again → tries to refresh again → loops forever. Flagged this to fix properly later
- The hero section needs to be full-bleed (100% viewport width) while the rest of the content is constrained. Solved by making `PageWrapper` full-width and having each section manage its own `container-pad`
- React Hook Form's `register()` spread and Zod `resolver` wiring took a while to get right the first time
- Getting the login redirect to remember where the user was going: `PrivateRoute` passes `state={{ from: location }}` to `<Navigate>`, and `LoginPage` reads `location.state?.from?.pathname`

---

## What I accomplished

- Full state management and API architecture in place
- Axios client with JWT interceptors built
- All 7 API modules, mock/real switching system, security utilities, and Zod schemas done
- Home page fully built with all 6 sections and real content
- Login, Register, Forgot Password pages fully functional with validation
- 6 reusable UI components that every other page will use

---

[← Day 4](day-4.md) · [Next: 29 June →](day-6.md)
