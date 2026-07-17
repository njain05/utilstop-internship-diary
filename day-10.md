# 03 July 2026 — Day 12: Silent Refresh on Page Load + End-to-End Verification

## What was the task?
Fix the reload-logout problem: when a user hits F5 or opens the app in a new tab, the JWT access token is lost from memory (Zustand is in-memory only), so they get logged out instantly. Implement a silent refresh that runs on every app load, checks if a valid session exists via the httpOnly cookie, and re-populates the Zustand store transparently.

---

## Why this happens

JWT access tokens are stored in Zustand — **in-memory only**, never in localStorage or sessionStorage. This is intentional security: hacker scripts running in the page (`XSS`) cannot steal a token they can't read from storage.

But the tradeoff is: JavaScript memory is wiped on every page reload. No token → user appears logged out → `PrivateRoute` redirects them to `/login`.

The fix: on every page load, **before rendering anything**, silently call `/auth/refresh`. The refresh endpoint reads the httpOnly cookie (which **does** survive reloads — it's stored by the browser, not JavaScript). If the cookie is valid, the server responds with a fresh access token, we store it in Zustand, and the user never notices a thing.

---

## What changed — 3 files

### 1. `src/api/auth.api.js` — refreshToken uses bare axios

The `refreshToken` function was using `apiClient`, which has the response interceptor. If the startup refresh fails (no valid cookie), the interceptor would trigger → try to refresh again → fail → redirect to `/login`. Wrong behavior for a startup check — we want a clean, silent failure.

**Before:**
```js
refreshToken: () => apiClient.post('/auth/refresh'),
```

**After:**
```js
refreshToken: () =>
  axios.post(`${env.API_BASE_URL}/auth/refresh`, {}, { withCredentials: true })
    .then((r) => r.data),
```

The `.then(r => r.data)` unwraps the axios response so callers get `{ accessToken }` directly — the same shape that the mock API returns. This keeps mock and real APIs consistent.

---

### 2. `src/hooks/useAuth.js` — set token BEFORE fetching profile

`refreshSession` called `authApi.refreshToken()` to get the new access token, then immediately called `authApi.getProfile()` to fetch the user's profile. But `getProfile()` uses `apiClient`, which reads the access token from the Zustand store — and the token wasn't in the store yet at that point.

Result: the profile request went out with no `Authorization` header and got a 401.

**Fix — set the token in the store first, then fetch the profile:**
```js
const { accessToken } = await authApi.refreshToken()
// Token in store now → getProfile() request will have the header
useAuthStore.getState().setTokenAndUser(accessToken, null)
const profile = await authApi.getProfile()
setTokenAndUser(accessToken, profile)
```

---

### 3. `src/main.jsx` — AuthInit wrapper component

Added a new `AuthInit` component that:
1. Runs `refreshSession()` exactly once on mount (a `useRef` flag prevents double-firing in React 18 StrictMode which intentionally mounts twice in development)
2. Shows a spinner while the check is in progress — this blocks `PrivateRoute` from flashing to `/login` before the check completes
3. Once the check resolves (success or failure), renders the full app

```jsx
function AuthInit({ children }) {
  const { refreshSession } = useAuth()
  const [ready, setReady] = useState(false)
  const attempted = useRef(false)

  useEffect(() => {
    if (attempted.current) return
    attempted.current = true
    const p = refreshSession()
    if (p) p.finally(() => setReady(true))
    else setReady(true)
  }, [refreshSession])

  if (!ready) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="w-8 h-8 rounded-full border-2 border-primary-500 border-t-transparent animate-spin" />
      </div>
    )
  }

  return children
}
```

---

## Bugs caught during verification

While testing, two wrong destructurings were found and fixed before the live run:

**Bug 1 — `client.js` interceptor (from yesterday's fix):**
```js
// Wrong — bare axios returns { data, status, headers }, not { accessToken }
const { accessToken } = await axios.post(...)

// Fixed
const { data: { accessToken } } = await axios.post(...)
```

**Bug 2 — `useAuth.js` mock mode mismatch:**
```js
// Wrong — assumed axios response wrapper, but mock returns { accessToken } directly
const { data: { accessToken } } = await authApi.refreshToken()

// Fixed — auth.api.js unwraps .data at source, so both mock and real return { accessToken }
const { accessToken } = await authApi.refreshToken()
```

---

## Live verification — Playwright end-to-end test

Ran a Playwright/Chromium browser session against the running dev server:

| Step | Result |
|------|--------|
| Page load | Spinner appeared immediately on first paint |
| AuthInit resolved | Spinner disappeared after ~200ms (mock refresh delay) |
| Page content | Home page rendered with "Nishtha Jain — Logout" visible — user authenticated |
| F5 / Reload | Spinner reappeared and resolved correctly — AuthInit re-runs each time |
| Direct `/account` visit | Stayed on `/account` — user was already authenticated after refresh |
| Console errors | 3 pre-existing CSP warnings (Google Fonts, frame-ancestors) — not caused by today's changes |
| Production build | `npm run build` — 263 modules, no errors, clean output |

---

## Problems faced

- React 18 StrictMode mounts components twice in development — without the `useRef` guard, `refreshSession` would fire twice on every page load, causing a double network call and a brief race condition
- The mock `refreshToken` returns `undefined` instead of a Promise when called while `isRefreshing` is already `true` — added a `if (p)` guard before calling `.finally()` to avoid a crash
- The three CSP console errors are pre-existing: `frame-ancestors` and `X-Frame-Options` can only be set as HTTP response headers, not `<meta>` tags. Google Fonts is blocked by the current `style-src` policy. These need to be moved to the web server config in a future task

---

## What I accomplished

- Silent refresh on page load — users no longer get logged out on F5
- `auth.api.js` and `useAuth.js` now have consistent and correct token handling
- Axois interceptor loop fix (from yesterday) is also correctly handling destructuring now
- Live end-to-end Playwright verification — confirmed in a real browser
- Production build is clean — 263 modules, zero errors

---

## What's next

The auth layer is now fully hardened. Next priority: build out the actual **booking flow step forms** — they're currently placeholders. That's the core feature of the entire product.

---

[← Day 9](day-9.md) · [Home](index.md)
