# 02 July 2026 — Day 11: Axios Interceptor Infinite Loop — Root Cause & Fix

## What was the task?
Investigate and fix the infinite loop risk in the Axios response interceptor inside `src/api/client.js`. Understand exactly where it happens and why, then refactor it safely.

---

## Background

The Axios client has a response interceptor that catches 401 errors and automatically refreshes the JWT token. The flow is:

```
Request fails with 401
  → interceptor fires
  → calls /auth/refresh to get a new token
  → retries the original request with the new token
```

The problem: the refresh call was made using `apiClient.post('/auth/refresh')` — the same Axios instance that has this very interceptor attached to it.

---

## The Bug — Infinite Loop

If `/auth/refresh` itself returns a 401 (e.g. the refresh token is expired, the user's session was revoked, the server restarted and cleared sessions), this is what happens:

```
apiClient.post('/auth/refresh') → 401
  → interceptor fires again
  → tries to call apiClient.post('/auth/refresh') again → 401
  → interceptor fires again
  → ... loops until the browser tab hangs
```

The `_retry` flag (`original._retry = true`) prevents the **original** failed request from looping, but does NOT protect the refresh call itself, because the refresh call is a new request with its own `config` object and no `_retry` flag.

---

## The Fix

Replace `apiClient.post('/auth/refresh')` with a **bare `axios.post()`** call — the underlying library, not our customized instance. Bare axios has no interceptors attached to it, so a 401 from `/auth/refresh` goes straight to the `catch` block, which then calls `logout()` and redirects to `/login`.

**Before:**
```js
const { accessToken } = await apiClient.post('/auth/refresh')
```

**After:**
```js
const { data: { accessToken } } = await axios.post(
  `${env.API_BASE_URL}/auth/refresh`,
  {},
  { withCredentials: true }   // still sends the httpOnly refresh cookie
)
```

Three things to note:
1. `axios` was already imported at the top of `client.js` — no new import needed
2. `env.API_BASE_URL` is already imported — so the full URL is constructed correctly
3. `withCredentials: true` is essential — without it, the browser won't send the httpOnly cookie that contains the refresh token

---

## What changed in `client.js`

| Line | Before | After |
|------|--------|-------|
| 48 | `apiClient.post('/auth/refresh')` | `axios.post(full_url, {}, { withCredentials: true })` |

One logical change. The rest of the file is untouched.

---

## Problems faced

- Initially easy to miss because the loop only triggers when the refresh token itself is invalid — which doesn't happen in normal usage or in mock mode. It's a silent time bomb that only fires in edge cases (session expiry, server restart, token revocation)
- Destructuring mismatch: bare `axios.post()` returns `{ data, status, headers, ... }` while `apiClient` (with the response interceptor) returns `data` directly. Had to use `{ data: { accessToken } }` destructuring instead of `{ accessToken }`

---

## What I accomplished

- Understood the full Axios interceptor flow inside out
- Identified and fixed the infinite loop risk
- One-line change with zero side effects — the rest of the auth flow is unchanged
- `src/api/client.js` is now safe against infinite refresh loops

---

[← 01 July](july-01.md) · [Next: 03 July →](july-03.md)
