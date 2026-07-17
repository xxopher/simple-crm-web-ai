---
name: reskin-app
description: Give this CRM app a new visual look (colors, type, shape, texture) by editing CSS only, without changing any component logic or structure. Use for requests like "reskin this app", "restyle it as X", "retheme it", "give it a new look/vibe/aesthetic", or "make it feel like Y".
---

This app is a Vite + React CRM. Its styling is fully decoupled from its logic, which is what makes it safe to restyle without reading or touching any component code:

- `src/index.css` — the design-token file. A `:root` block of CSS custom properties (colors, type scale, spacing, radii, shadows, motion) that every component consumes via `var(--token-name)`. This is the starting lever, but a full reskin also means visiting every module below — see the process.
- `src/App.css` — shared classes used across pages (e.g. page headers, primary buttons, back links).
- One CSS Module per component/page, each scoped to that file only:
  `src/components/{CustomerCard,CustomerDetail,Header,SearchBar,Sidebar,Spinner}.module.css`,
  `src/layouts/RootLayout.module.css`,
  `src/pages/{CustomerDetailPage,DashboardPage,EditCustomerPage,LoginPage,WelcomePage}.module.css`.
- `index.html` — may get a `<link>` for a Google Font, or a self-hosted `@font-face`, if the new look needs one.

There is currently only **one theme** (no dark-mode tokens exist yet) — don't assume a `prefers-color-scheme` or `[data-theme]` split is already there. Add one only if asked for a dark mode; otherwise a single cohesive theme is correct.

## Hard constraints

- Never edit any `.jsx` or `.js` file. If achieving the request seems to require a structural or behavioral change, stop and say so instead of touching logic.
- Never rename or remove an existing `--custom-property`. Change its *value*, or add new tokens — components reference these names directly, so a rename silently breaks them.
- Never invent a new class name that isn't already used by the component you're styling. Restyle what's there.
- `index.html` may only change to add font loading (a `<link>` tag or a `<style>` block with `@font-face`) — nothing else.

## Process

1. Get the theme/mood in one sentence (ask if it wasn't given). Pick a real, named palette — 4–6 hex values — and map them onto the *existing* token names in `src/index.css` (`--neutral-*`, `--primary-*`, `--success-*`, `--danger-*`, and the semantic aliases below them). Don't just swap one accent color; commit to the palette across neutrals and semantics too, or the result will look like a tint, not a reskin.
2. Pick a deliberate type pairing for `--font-sans`/`--font-mono` (and add a display face via Google Fonts/`@font-face` in `index.html` if the theme calls for one distinct from body text). Adjust `--text-*` sizes only if the new type needs more/less room to read well.
3. Decide the shape and texture language in one line — corner radius (`--radius-*`), border weight, shadow style (`--shadow-*`) — and apply it consistently rather than leaving some components on the old defaults.
4. Apply changes in this order: `src/index.css` first (this alone reskins most of the app since components read from it), then `src/App.css`. This carries the theme most of the way, but doesn't finish it — treat it as the palette, not the reskin.
5. Then go through **every** CSS Module file listed above, one by one, and bring each in line with the new theme. This is what makes the result cohesive rather than a re-tinted version of the old design. In each file, look for:
   - hardcoded colors, radii, or shadows that bypass the tokens (literal `white`, `#fff`, `0 1px 2px rgba(...)`, a stray `border-radius: 8px`) — replace with the relevant `var(--token)` so they pick up the new theme too;
   - borders, dividers, and shadows that should get the new weight/treatment even if they were already using tokens whose *values* changed but whose usage (e.g. `border: 1px solid` where the theme wants `2–3px`) didn't;
   - labels, badges, or headings that would read better with the new display font, letter-spacing, or case treatment (e.g. uppercase eyebrows) than plain body text;
   - any state (hover/active/selected) that was drawn with a hardcoded value instead of a token, which would otherwise silently stay on the old theme.
   Every module file should end up touched or explicitly confirmed as already token-driven and therefore fine as-is — don't skip files because they "probably already look okay."
6. Keep interaction states honest — hover, focus-visible (`--ring`), and disabled states should still be visibly distinct after the reskin, not just recolored to blend in.

## Before finishing

- Run `git diff --stat` and confirm only CSS files and/or `index.html` changed — if anything else shows up, that's a bug, revert it.
- Check the diff covers all 12 module files, `App.css`, and `index.css` — a cohesive reskin touches (or explicitly confirms as already fine) every one of them, not just the token file.
