# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

| Command | Action |
|---|---|
| `npm run dev` | Start dev server at `localhost:4321` |
| `npm run build` | Build to `./dist/` |
| `npm run preview` | Preview production build locally |
| `npm run astro add <integration>` | Add integrations (react, tailwind, etc.) |
| `npm run astro check` | Type-check `.astro` files |

The preview server is configured in `.claude/launch.json` (name: `tuition-dev`, port: 4321). Use `mcp__Claude_Preview__preview_start` with that name to launch it in-session.

## Project

**Online Tuition Classes** — a frontend-only marketing site for an online tutoring service matching students (Class 1–12) with verified teachers. No backend or database. The "Find a Teacher" form submits to **Formspree** (`https://formspree.io/f/mpqgpdjn`) and emails `abhinamika@gmail.com`; the contact page form is still client-side with a success-state toggle.

This is a **multi-page application (MPA)**. Pages and their URLs:

| File | URL | Purpose |
|---|---|---|
| `src/pages/index.astro` | `/` | Home — hero, How It Works, Pricing calculator, Find a Teacher form |
| `src/pages/about.astro` | `/about` | Mission, verification process, stats |
| `src/pages/contact.astro` | `/contact` | Contact info + message form |
| `src/pages/profile.astro` | `/profile` | User profile — sign up / log in / edit details |
| `src/pages/privacy-policy.astro` | `/privacy-policy` | Privacy policy (7 sections, sticky TOC) |
| `src/pages/terms.astro` | `/terms` | Terms & conditions (9 sections, sticky TOC) |

## Architecture

This is an [Astro](https://docs.astro.build) project using TypeScript (strict mode).

- `src/pages/` — file-based routing; each `.astro` file is a URL
- `src/layouts/Layout.astro` — HTML shell: `<head>`, global resets, Inter font, and the **Lenis smooth-scroll bootstrap** (`<script>` at the bottom of `<body>`)
- `src/components/Nav.astro` — shared sticky nav; accepts `activePage?: string` prop. Has a circular icon-only profile button (no text). On mobile (≤640px) nav links collapse into a hamburger drawer; Privacy + Terms links hidden on mid-size screens (641–768px) and shown on ≥769px.
- `src/components/Footer.astro` — shared dark footer with Company / Legal / Students link columns
- `src/assets/` — images processed by Astro's asset pipeline
- `public/` — static files served as-is

Pages import `Layout`, `Nav`, and `Footer`, then compose their content between them. Frontmatter (`---` fences) runs server-side at build time.

**Page transitions** use Astro's `<ClientRouter />` (from `astro:transitions`, available in Astro 5+; called `ViewTransitions` in older versions). It's imported and placed in the `<head>` inside `Layout.astro`. This enables SPA-style navigation with a fade+scale animation defined via `::view-transition-old/new(root)` in the global CSS. It also auto-prefetches linked pages. After each navigation `astro:after-swap` fires — use this event to re-initialise any page-level JS (Lenis, hamburger, nav auth state all subscribe to it).

**Smooth scrolling** is handled by [Lenis](https://github.com/darkroomengineering/lenis) (`lenis` npm package, `autoRaf: true`), initialised in `Layout.astro` and re-initialised on `astro:after-swap`. Do not add `scroll-behavior: smooth` to CSS — Lenis handles it.

**Styles** are colocated `<style>` blocks inside each `.astro` file using CSS custom properties (`--ink`, `--canvas`, etc.). There is no global stylesheet beyond the reset in `Layout.astro`. Token definitions are repeated per-page because Astro scopes component styles.

**Scroll-reveal animations**: `Layout.astro` includes a global `IntersectionObserver` script (also re-initialised on `astro:after-swap`). Add `class="reveal"` to any element to fade+slide it in when it enters the viewport. Add `class="reveal-stagger reveal"` to a container to stagger-animate its direct children with cascading delays (supports up to 6 children). Global CSS for both classes lives in the `<style is:global>` block in `Layout.astro`.

**Astro CSS scoping gotcha**: styles in `<style>` blocks are scoped by adding a `data-astro-cid-*` attribute to elements in the template. Elements created dynamically via JavaScript (`innerHTML`, `createElement`) do NOT receive this attribute, so scoped selectors won't match them. Use `:global(.classname)` for any styles targeting JS-rendered elements.

**Specificity trap with utility classes**: a scoped utility like `.hidden { display: none }` has the same specificity as `.auth-form { display: flex }` after Astro adds the `[data-astro-cid-*]` attribute to both. If `.auth-form` appears later in the stylesheet it will win, leaving `.hidden` ineffective. Always declare `.hidden { display: none !important }` to guarantee it takes priority.

## Auth system

Frontend-only auth using **`localStorage`** (key `otc_user`, JSON `{name, email, phone, picture}`). No backend — email is the identifier. Supports two sign-in methods: email-only (no password) and Google OAuth.

- **`Nav.astro`** — reads `otc_user` on load and listens for both `otc-auth-change` and `astro:after-swap` (to re-run after page transitions). When logged in: shows the Google profile photo (`picture` field) if present, otherwise falls back to the user's first initial. The profile button is always a 36px circle. The hamburger open/close logic also re-initialises on `astro:after-swap`.
- **`index.astro`** — the "Find a Teacher" form submit handler checks for `otc_user`; if absent it opens the auth modal (`#auth-modal`) instead of submitting. The modal has both email forms and a Google Sign-In button. After any successful login/signup, `submitFindForm()` is called automatically. `saveUser()` dispatches `otc-auth-change` so the nav updates immediately.
- **`profile.astro`** — standalone login/signup/edit/logout page. Same `otc_user` key and `otc-auth-change` event. Accepts `?tab=signup` in the URL to open the signup tab directly. The profile avatar shows the Google photo when `picture` is set.

**Google Sign-In** uses [Google Identity Services](https://developers.google.com/identity/gsi/web) (`https://accounts.google.com/gsi/client`). The Client ID (`945888240197-9in6fs6u0gmtuib5b3de70ek7ceam4j4.apps.googleusercontent.com`) is hardcoded as `GOOGLE_CLIENT_ID` in both `index.astro` and `profile.astro`. The GSI callback decodes the returned JWT with `atob()` to extract `name`, `email`, and `picture` — no server-side verification needed for this use case. When buying the production domain (`onlinetuitionclasses.com`), add it to the authorized JavaScript origins in Google Cloud Console.

The three helper functions (`getUser`, `saveUser`, `removeUser`) are **duplicated** in `index.astro` and `profile.astro` because Astro `<script is:inline>` blocks don't share scope across pages. Do not try to deduplicate into a shared module without switching to `<script>` (bundled) mode.

## Pricing Calculator (homepage)

The interactive pricing section in `src/pages/index.astro` is driven entirely by `<script is:inline>` — no framework. Key logic:

- **Class slider** (1–12): drag handle updates a fill bar and badge in real time. No CSS transition on the fill — updates synchronously via `requestAnimationFrame` to avoid lag.
- **Subjects**: Maths, Physics, Chemistry, Biology. Max 2 selectable. **Maths and Biology are hidden for class 11–12** (only available up to class 10). Subject cards are rendered by JS into `#subject-grid`; their styles use `:global()` for this reason.
- No price is displayed — the calculator captures class + subject selection for the "Find a Teacher" form flow.
- **Shared state**: `currentClass` (number) and `calcSelected` (array of subject name strings) are declared as `var` in the outer `<script is:inline>` scope so the form submit handler can read them. The IIFE that drives the calculator writes to these outer vars. Do not move them inside the IIFE.
- **Form validation**: the "Find a Teacher" form submit handler checks `calcSelected.length > 0` before proceeding. If empty it shows `#calc-error` (amber banner with shake animation) and blocks submission. The error auto-hides when the user selects a subject. On successful Formspree submission the form is replaced by `#form-success` — an animated card with SVG checkmark that draws itself in via `stroke-dashoffset` animation.

## Design System

UI work must follow `DESIGN.md` (Vercel-inspired design language). Key rules:

- **Colors**: Near-white `#fafafa` canvas, ink-near-black `#171717` for primary CTAs and dark bands, `#4d4d4d` for secondary text.
- **Typography**: Geist (geometric sans) at weights 400/500/600 — never 700+. **Current implementation uses Inter** (Google Fonts) as a substitute; swap to Geist when self-hosted. Headlines are sentence-case with aggressive negative letter-spacing.
- **Buttons**: 100px pill radius for marketing CTAs, 6px radius for nav-scale buttons. Black-fill primary, white-fill secondary.
- **Cards**: Stacked multi-offset shadows + 1px inset hairline border — never a single heavy drop-shadow.
- **Surfaces**: Band through `canvas-soft` (#fafafa) → `canvas` (#ffffff) → `primary` (#171717 dark band).
- **Spacing**: 4px base unit. Section padding 64–96px vertical; hero bands 192px.
- **Inner page hero**: dark `#171717` band with mono-caps eyebrow, sentence-case headline, muted subtext — see `about.astro` as reference.
- **Legal page pattern**: two-column layout with sticky sidebar TOC + prose body; sidebar hidden on mobile — see `privacy-policy.astro` as reference.
