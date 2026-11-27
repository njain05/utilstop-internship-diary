# 04 July 2026 — Day 11: Supervisor Meeting + OTP Auth + Real API Wiring + Static Pages

## What was the task?

Today had two parts: a supervisor meeting in the morning, and a big implementation sprint after it based on what was decided.

---

## The Meeting (11:07 IST with Mani)

Met with the supervisor to clarify the real API structure, auth flow, and product requirements before building further.

### Key decisions made

**Auth — OTP only, not email/password:**
The app uses phone-based OTP authentication, same as the mobile app. There is no email/password login at all. The flow is: enter phone number → receive SMS OTP → enter OTP → get JWT token.

- `POST /generateOTP` — takes `mobileNumber` and `is_mobile: true`, returns `user_id`, `role`, `twilioService_id`, `is_profileCreated`
- `POST /verifyOTP` — takes those values + the OTP the user entered, returns `token` and `user_details`
- New users are auto-created when `is_mobile: true` is passed and the number isn't found
- `is_mobile: false` is only for admin/vendor roles — passing it for end users returns "No user found"

**Token storage — localStorage:**
Supervisor confirmed the token goes in `localStorage`. The token is valid for 360 days, so there is no refresh cycle needed for now. Key names: `ut_token` (the JWT), `ut_user` (the user object JSON).

**Booking flow — 4-step, mirroring mobile app:**
The booking flow must mirror the mobile app's four steps: location selection → date/time selection → service offerings selection → review and payment. This is the next major feature to build after today.

**Static info pages to build:**
Partner With Us, Contact Us, Career, FAQ, Terms & Conditions, Privacy Policy — all content can be found in the API engine's views folder.

**All auth routes are at root `/api/v1/`:**
No `/auth/` prefix. So it's `/api/v1/generateOTP`, not `/api/v1/auth/generateOTP`.

**Hardcoded test numbers (bypass Twilio, any OTP works):**
- `91XXXXXXXXXX` (India)
- `91XXXXXXXXXX` (India)
- `971XXXXXXXXX` (UAE)

---

## What we built after the meeting

### 1. Fixed CORS — Vite proxy

Every API call was failing with a CORS error because the browser blocks requests from `localhost` directly to `https://<development-api-server>`. Fixed by changing all Axios `baseURL` values to `/api` (relative) and adding a Vite proxy in `vite.config.js`:

```js
proxy: {
  '/api': {
    target: 'https://<development-api-server>',
    changeOrigin: true,
    rewrite: (p) => p.replace(/^\/api/, '/api/v1'),
  },
}
```

The browser sends requests to `localhost/api/...`, the Vite dev server forwards them server-to-server to the live API — no CORS restriction applies to server-to-server calls.

---

### 2. Rebuilt auth — OTP flow

Rewrote `src/api/auth.api.js` completely. Removed email/password, replaced with:

```js
generateOTP: (mobileNumber) =>
  apiClient.post('/generateOTP', { mobileNumber, is_mobile: true }),
verifyOTP: (payload) =>
  apiClient.post('/verifyOTP', payload),
updateProfile: (data) =>
  apiClient.post('/newUpdateUser', data),
getProfile: () =>
  apiClient.get('/myProfile'),
```

Updated `src/store/authStore.js` to hydrate from localStorage on init and save token + user on login:

```js
const TOKEN_KEY = 'ut_token'
const USER_KEY = 'ut_user'

// On init — reads from localStorage so token survives page reload
accessToken: localStorage.getItem(TOKEN_KEY) ?? null,
user: loadUser(),
isAuthenticated: !!localStorage.getItem(TOKEN_KEY),

// On login — saves to localStorage
setTokenAndUser: (token, user) => {
  localStorage.setItem(TOKEN_KEY, token)
  localStorage.setItem(USER_KEY, JSON.stringify(user))
  set({ accessToken: token, user, isAuthenticated: !!token })
},
```

Updated `src/hooks/useAuth.js` with `sendOtp`, `verifyOtp`, and `setupProfile` methods.

---

### 3. Built 3 new auth pages

**`PhoneLoginPage.jsx`** — Country selector (+971 UAE / +91 India), phone number input, calls `sendOtp`, passes the OTP state to the next page via React Router's `navigate(..., { state: {...} })`.

**`OtpVerifyPage.jsx`** — 6-box OTP input, auto-advances on each digit, supports paste, keyboard backspace navigation. Calls `verifyOtp` with the full payload from the previous page. Routes to `/setup-profile` if the account is new, otherwise to home.

**`ProfileSetupPage.jsx`** — Simple name input for brand-new accounts. Calls `updateProfile`, stores updated user in the store, redirects home.

---

### 4. Wired services to the real API shape

Fixed `ServiceCard.jsx`, `ServiceListPage.jsx`, `HomePage.jsx`, `ServiceDetailPage.jsx`, and `BookingFlowPage.jsx` to use the real API field names:

- `service.name` (not `displayName`)
- `service.mobile_redirect` as the URL slug
- `service.image_url` for the image
- Badges: `highlight`, `quick_services`, `ai_price`

`useServices()` and `useServiceBySlug()` hooks now use `publicApi` (a separate Axios instance with a static Bearer token) so the services grid loads without requiring the user to be logged in.

---

### 5. Fixed the header

Header was showing "Account" instead of the user's name. Root cause: the API returns `first_name`, but the header was reading `user?.name`. Fixed to `user?.first_name ?? 'Account'`. Added a red avatar circle showing the first letter.

---

### 6. Built all 6 static/info pages

Built from scratch using content from the API engine's views folder:

| Page | Route | Notes |
|---|---|---|
| Contact Us | `/contact` | Form: name, email, mobile, message. Success banner (reCaptcha not set up yet) |
| FAQ | `/faq` | 8-item accordion, one open at a time |
| Career | `/career` | Role accordion + application form that posts to `/api/career_inquiry` with `{name, mobile, email, job_role, job_title}` |
| Partner With Us | `/partner` | Benefits cards + registration form (success banner, no real API — reCaptcha required) |
| Terms & Conditions | `/terms` | 8 static sections from the real API engine content |
| Privacy Policy | `/privacy` | 6 static sections |

All 6 pages are wired into `AppRouter.jsx` and all nav links in the header point to them.

---

## Problems faced

**"authApi.generateOTP is not a function"** — The `.env` file had `VITE_MOCK_API=true`, so the mock auth API (which doesn't have `generateOTP`) was being loaded. Fixed by hardcoding `export const authApi = realAuthApi` in `src/api/index.js`.

**"Cannot POST /api/v1/auth/generateOTP" (404)** — Auth routes are mounted at `/api/v1/` directly, not under `/api/v1/auth/`. Removed the `/auth/` prefix from all auth paths.

**Double country code** — If the user selects +91 and also types "91XXXXXXXXXX", the number becomes "9191XXXXXXXXXX". Added a note in the UI placeholder to enter just the digits without the country code.

**"No user found" from generateOTP** — Was passing `is_mobile: false` initially. For end users, `is_mobile: true` is required — it also auto-creates the account if the number is new.

---

## What I accomplished

- CORS completely resolved — all API calls now work from localhost
- Full OTP auth flow — phone login, OTP verify, profile setup, all working end-to-end
- Token persists across page reloads (localStorage, per supervisor instruction)
- Services grid now shows real data from the live API
- Header shows the logged-in user's name with an avatar
- All 6 static/info pages built and routed
- Production build: 262 modules, 0 errors

---

## What's next

The biggest remaining task: **4-step booking flow** — location → date/time → service offerings → review & payment. Currently the `BookingFlowPage` just shows a placeholder. Per the meeting, this must mirror the mobile app's flow exactly.

---

[← Day 10](day-10.md) · [Home](index.md) · [Day 12 →](day-12.md)
