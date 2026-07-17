# 24 June 2026 — Day 3: Tech Stack Decision

## What was the task?
Research and finalize the tech stack. Install all dependencies. Understand why each library is chosen and what role it plays.

---

## What did we do?

- Researched modern React project setups
- Finalized the full tech stack:

| Layer | Choice | Why |
|-------|--------|-----|
| UI framework | React 18 | Industry standard, component model fits the service card grid |
| Build tool | Vite | Much faster than CRA, excellent HMR |
| Styling | Tailwind CSS | Utility-first — fast to write, consistent spacing |
| Routing | React Router v6 | Nested routes fit the booking flow perfectly |
| Server state | TanStack Query v5 | Caching, background refetch, loading states out of the box |
| Client state | Zustand | Lightweight, no boilerplate — perfect for auth token + booking state |
| Forms | React Hook Form + Zod | Performance forms + runtime schema validation in one combo |
| HTTP | Axios | Interceptors needed for automatic JWT refresh |
| Security | DOMPurify | Sanitize user input, especially the bid amount field |

- Created `package.json` with all dependencies
- Ran `npm install` — first time seeing the node_modules folder appear

---

## Problems faced

- Confused between Zustand and Redux at first — decided Zustand because Redux felt like too much boilerplate for a project this size
- TanStack Query v5 has a different API than v4 — had to read the new docs carefully (the `useQuery` options changed)
- Zod and React Hook Form need a resolver package (`@hookform/resolvers`) to work together — easy to miss

---

## What I accomplished

- Tech stack finalized and documented
- All packages installed successfully
- Understood the role of every library in the project

---

[← 23 June](june-23.md) · [Next: 25 June →](june-25.md)
