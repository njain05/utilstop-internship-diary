# 25 June 2026 — Day 4: Vite Scaffold + Design System

## What was the task?
Initialize the Vite + React project. Configure Tailwind CSS with the UtilStop brand colors and design tokens from Figma. Get a "Hello World" rendering in the browser.

---

## What did we do?

- Ran `npm create vite@latest` with the React template
- Installed and configured Tailwind CSS with PostCSS
- Set up `tailwind.config.js` with the full UtilStop design system:
  - Primary color scale based on `#C81849`
  - Neutral scale for text and borders
  - Semantic scales: `error`, `success`, `warning`
  - Custom typography utilities (`text-title`, `text-para-lg`, etc.)
  - Custom shadow tokens (`shadow-card`, `shadow-card-hover`)
- Set up `src/index.css` with:
  - Google Inter font import
  - CSS custom properties for brand colors
  - `.container-pad` utility (max-width 1280px with responsive padding)
- Set up `vite.config.js` with:
  - `@/` path alias pointing to `src/`
  - Brotli compression plugin for production builds
- Configured `.env` and `.env.example` with `VITE_API_BASE_URL` and `VITE_MOCK_API`

---

## Problems faced

- Tailwind's default color palette clashes with custom color names — had to use `extend` properly so I didn't wipe out built-in utilities
- The `@/` path alias needs to be set in both `vite.config.js` AND a `jsconfig.json` (or `tsconfig`) for the editor to understand it — forgot the editor config at first and got red squiggles everywhere
- Google Fonts in `index.css` gets blocked by the Content Security Policy we set in `index.html` — noted as a known issue to fix later when we move headers to the server

---

## What I accomplished

- Vite project running at `localhost:5173`
- Full design token system configured in Tailwind
- Brand colors, typography, and shadows all working
- Path alias `@/` working for clean imports
- Environment variable system in place

---

[← 24 June](june-24.md) · [Next: 26 June →](june-26.md)
