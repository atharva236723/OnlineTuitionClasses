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

When running the dev server as an agent (not interactively), use background mode: `npx astro dev &` — then stop it with `kill %1` when done. On Windows PowerShell use `Start-Process` or run via Bash tool instead, since `kill %1` is a bash job-control syntax.

## MCP servers & skills

- **`astro-docs`** — Astro documentation MCP, HTTP transport at `https://mcp.docs.astro.build/mcp`. Use for Astro API questions, component syntax, and integration docs.
- **`web-design-guidelines`** skill — installed at `.agents/skills/web-design-guidelines`. Invoke via `/web-design-guidelines` to audit UI code against the Vercel-inspired design language in `DESIGN.md`.

## Git branches

The local working branch is `master`. The GitHub default branch is `main`. Always push with:

```sh
git push origin master:main
```

or push to both if needed: `git push origin master` then `git push origin master:main`.

## Project

**Online Tuition Classes** — a frontend-only marketing site for an online tutoring service matching students (Class 1–12) with verified teachers. No backend or database. All user-facing forms submit to **Formspree** (`https://formspree.io/f/mpqgpdjn`) and email `abhinamika@gmail.com`. Each form includes a hidden `Type` field so submissions are distinguishable in the inbox:

| Form | `Type` value |
|---|---|
| Find a Teacher (homepage) | *(none)* |
| Batch Registration (homepage) | `Batch Registration` |
| Contact page | `Contact Message` |
| Profile signup | `New User Signup` |

This is a **hybrid MPA/SPA**: Astro generates a separate HTML file per route (MPA), but `<ClientRouter />` is enabled globally so navigation is client-side with DOM swapping (SPA-style). Pages and their URLs:

| File | URL | Purpose |
|---|---|---|
| `src/pages/index.astro` | `/` | Home — hero, How It Works, Smart Match Quiz, Pricing calculator, Find a Teacher form, Batch Registration, FAQ, SEO FAQ (`#seo-faq`), SEO content (`#about-otc`) |
| `src/pages/about.astro` | `/about` | Mission and informational sections (no stats) |
| `src/pages/contact.astro` | `/contact` | Contact message form (no address/phone displayed) |
| `src/pages/profile.astro` | `/profile` | User profile — sign up / log in / edit details |
| `src/pages/privacy-policy.astro` | `/privacy-policy` | Privacy policy (7 sections, sticky TOC) |
| `src/pages/terms.astro` | `/terms` | Terms & conditions (8 sections, sticky TOC); section 5 includes non-refundable fee and compulsory attendance policy |
| `src/pages/interview-prep/index.astro` | `/interview-prep` | Interview Prep hub — hero with animated stat counters, 5 module navigation cards, CTA |
| `src/pages/interview-prep/compare.astro` | `/interview-prep/compare` | Module 1 — 3D flip-card Mediocre vs Pro (8 comparisons) + Common Mistakes grid (8 cards) |
| `src/pages/interview-prep/frameworks.astro` | `/interview-prep/frameworks` | Module 2 — STAR / CAR / SOAR answer frameworks with step-by-step cards + quick-reference table |
| `src/pages/interview-prep/questions.astro` | `/interview-prep/questions` | Module 3 — 15 Q&As (3 tabbed categories) with mediocre/pro answers + 30+ question practice bank |
| `src/pages/interview-prep/plan.astro` | `/interview-prep/plan` | Module 4 — 7-step 14-day countdown timeline + localStorage-persisted prep checklist (12 items, category-tagged) |
| `src/pages/interview-prep/videos.astro` | `/interview-prep/videos` | Module 5 — tabbed YouTube video hub with modal player (4 categories, 8 curated videos) |
| `src/pages/404.astro` | `/*` (unmatched) | 404 error page — dark full-viewport layout, auto-served by the host on missing routes |

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

**Smooth scrolling** is handled by [Lenis](https://github.com/darkroomengineering/lenis) (`lenis` npm package, `autoRaf: true`), initialised in `Layout.astro` and re-initialised on `astro:after-swap`. Do not add `scroll-behavior: smooth` to CSS — Lenis handles it. Lenis also intercepts all `a[href^="#"]` anchor clicks and calls `lenis.scrollTo(target, { offset: -72 })` — the `-72` offset accounts for the sticky nav height so headings aren't hidden behind it. Do not remove this offset.

**Important**: `window.lenis` is the Lenis **npm package object**, not the instance — calling `window.lenis.scrollTo(...)` will fail. The actual Lenis instance is exposed as `window.lenisInstance` (set in `Layout.astro` after `new Lenis(...)`). Any `<script is:inline>` that needs to programmatically scroll should use `window.lenisInstance.scrollTo(element, { offset: -72 })` with a fallback to `element.scrollIntoView()`.

**Styles** are colocated `<style>` blocks inside each `.astro` file using CSS custom properties. The canonical token definitions live in the `<style is:global>` block of `Layout.astro` under `:root` — do **not** re-declare them in per-page `<style>` blocks (they were previously duplicated per-page; that pattern is removed). Dark-mode overrides live under `[data-theme="dark"]` in the same global block.

**Scroll-reveal animations**: `Layout.astro` includes a global `IntersectionObserver` script (also re-initialised on `astro:after-swap`). Add `class="reveal"` to any element to fade+slide it in when it enters the viewport. Add `class="reveal-stagger reveal"` to a container to stagger-animate its direct children with cascading delays (supports up to 6 children). Global CSS for both classes lives in the `<style is:global>` block in `Layout.astro`.

**Dark / light mode**: Toggled by `data-theme="dark"|"light"` on `<html>`. A FOUC-preventing inline `<script>` in `<head>` (inside `Layout.astro`) reads `localStorage.getItem('otc-theme')` or `prefers-color-scheme` and sets the attribute before first paint. The theme toggle button in `Nav.astro` (id `theme-toggle`, always visible including on mobile) writes back to `localStorage` under key `otc-theme` and updates the attribute. The `[data-theme="dark"]` block in `Layout.astro` overrides every token — it wins over per-page `:root` rules because attribute selectors (0,1,0) beat pseudo-class selectors (0,0,1).

Global design tokens:

| Token | Light | Dark | Usage |
|---|---|---|---|
| `--ink` | `#171717` | `#ededed` | Primary text; also backgrounds of buttons/badges/avatars (pair with `--canvas` for text on those) |
| `--ink-2` | `#4d4d4d` | `#a3a3a3` | Secondary / muted text |
| `--canvas` | `#ffffff` | `#141414` | Card / modal backgrounds |
| `--canvas-soft` | `#fafafa` | `#1c1c1c` | Alternate section backgrounds |
| `--border` | `#e5e5e5` | `#2f2f2f` | All borders |
| `--bg` | `#fafafa` | `#0f0f0f` | Page `<html>` background |
| `--surface-dark` | `#171717` | `#0a0a0a` | Dark band sections (hero, quiz) — stays near-black in both modes |
| `--nav-bg` | `rgba(255,255,255,0.92)` | `rgba(15,15,15,0.92)` | Sticky nav background |
| `--mobile-menu-bg` | `rgba(255,255,255,0.97)` | `rgba(15,15,15,0.97)` | Mobile drawer background |

Key rules:
- Dark band sections (hero, `.section-dark`, page-hero) use `background: var(--surface-dark)` — **not** `var(--ink)`.
- Buttons/badges with `background: var(--ink)` must use `color: var(--canvas)` (not `#fff`) so they get dark text in dark mode.
- The footer (`Footer.astro`) is hardcoded `#171717` — it is intentionally always dark and does not participate in the theme system.
- Interview Prep pages use their own `--ip-*` token set (dark `#0a0a0a` base) and do not need light/dark theming.

**Astro CSS scoping gotcha**: styles in `<style>` blocks are scoped by adding a `data-astro-cid-*` attribute to elements in the template. Elements created dynamically via JavaScript (`innerHTML`, `createElement`) do NOT receive this attribute, so scoped selectors won't match them. Use `:global(.classname)` for any styles targeting JS-rendered elements.

**Specificity trap with utility classes**: a scoped utility like `.hidden { display: none }` has the same specificity as `.auth-form { display: flex }` after Astro adds the `[data-astro-cid-*]` attribute to both. If `.auth-form` appears later in the stylesheet it will win, leaving `.hidden` ineffective. Always declare `.hidden { display: none !important }` to guarantee it takes priority.

## SEO system

`Layout.astro` accepts the following optional SEO props — pass them from each page to override the defaults:

| Prop | Default | Purpose |
|---|---|---|
| `title` | `"Online Tuition Classes for Class 1–12 \| Verified Teachers"` | `<title>` tag |
| `description` | Site-wide default | `<meta name="description">` |
| `keywords` | Site-wide keyword list | `<meta name="keywords">` |
| `ogTitle` | Falls back to `title` | `og:title` |
| `ogDescription` | Falls back to `description` | `og:description` |
| `ogImage` | `/og-image.png` | `og:image` and `twitter:image` |
| `ogUrl` | `https://onlinetuitionclasses.com` | `og:url` |
| `canonical` | *(none)* | `<link rel="canonical">` — set on every indexable page |

Twitter Card is always `summary_large_image`. Every page should pass `canonical` and `ogUrl` pointing to its own absolute URL.

**JSON-LD FAQ schema** — `index.astro` includes a `<script type="application/ld+json">` block with `@type: FAQPage` and 16 Q&As covering online-classes and interview-prep keywords. This is placed directly in the page template (not in Layout) so it only appears on the homepage. When adding FAQ structured data to other pages, follow the same inline pattern — do not put JSON-LD in `Layout.astro`.

**Homepage SEO sections** (below the existing FAQ accordion):
- `#seo-faq` — visible 16-item accordion FAQ targeting long-tail keyword queries; styled with `.seo-faq-*` classes defined in the page `<style>` block.
- `#about-otc` — ~600-word SEO prose section with H3 subheadings; styled with `.seo-*` classes. Uses `var(--canvas-soft)` background.

## Auth system

Frontend-only auth using **`localStorage`** (key `otc_user`, JSON `{name, email, phone, picture}`). No backend — email is the identifier. Supports two sign-in methods: email-only (no password) and Google OAuth.

- **`Nav.astro`** — reads `otc_user` on load and listens for both `otc-auth-change` and `astro:after-swap` (to re-run after page transitions). When logged in: shows the Google profile photo (`picture` field) if present, otherwise falls back to the user's first initial. The profile button is always a 36px circle. The hamburger open/close logic also re-initialises on `astro:after-swap`.
- **`index.astro`** — the "Find a Teacher" form submit handler checks for `otc_user`; if absent it opens the auth modal (`#auth-modal`) instead of submitting. The modal has both email forms and a Google Sign-In button. After any successful login/signup, `submitFindForm()` is called automatically. `saveUser()` dispatches `otc-auth-change` so the nav updates immediately.
- **`profile.astro`** — standalone login/signup/edit/logout page. Same `otc_user` key and `otc-auth-change` event. Accepts `?tab=signup` in the URL to open the signup tab directly. The profile avatar shows the Google photo when `picture` is set.

**Google Sign-In** uses [Google Identity Services](https://developers.google.com/identity/gsi/web) (`https://accounts.google.com/gsi/client`). The Client ID (`945888240197-9in6fs6u0gmtuib5b3de70ek7ceam4j4.apps.googleusercontent.com`) is hardcoded as `GOOGLE_CLIENT_ID` in both `index.astro` and `profile.astro`. The GSI callback decodes the returned JWT with `atob()` to extract `name`, `email`, and `picture` — no server-side verification needed for this use case. When buying the production domain (`onlinetuitionclasses.com`), add it to the authorized JavaScript origins in Google Cloud Console.

The three helper functions (`getUser`, `saveUser`, `removeUser`) are **duplicated** in `index.astro` and `profile.astro` because Astro `<script is:inline>` blocks don't share scope across pages. Do not try to deduplicate into a shared module without switching to `<script>` (bundled) mode.

## Homepage-only sections (index.astro)

**Smart Match Quiz** (`#quiz`) — dark `#171717` band section between "How It Works" and Pricing. A **2-step** interactive quiz (Goal → Learning Style) driven by a separate `<script is:inline>` block at the bottom of the file (after `</Layout>`). Answers are stored in a local `answers` object. After step 2, a **loading/searching animation** plays (`#qstep-loading`) for ~5 seconds — three checklist rows tick off sequentially, the label changes to "Match found!", then the result panel (`#qstep-result`) is shown with a "Find this teacher →" CTA that smooth-scrolls to `#find`. Progress dots are hidden during the loading and result steps. The quiz (including loading state) resets fully when "Retake quiz" is clicked via `resetLoadingState()`. The quiz re-initialises on `astro:after-swap` via `document.addEventListener('astro:after-swap', initQuiz)`.

**FAQ** (`#faq`) — uses native HTML `<details>`/`<summary>` accordion. The `+` → `×` rotation is pure CSS via `.faq-item[open] .faq-q::after { transform: rotate(45deg) }`. No JS needed.

**SEO FAQ** (`#seo-faq`) — a second, larger `<details>`/`<summary>` accordion below the existing FAQ, targeting 16 long-tail keyword queries across online classes and interview prep. Styled with `.seo-faq-*` classes (card border layout, `var(--canvas)` / `var(--canvas-soft)` backgrounds). Accompanied by a `<script type="application/ld+json">` block immediately after the section with `@type: FAQPage` schema (16 entries). Do not move the JSON-LD into `Layout.astro` — it should only appear on this page.

**SEO content** (`#about-otc`) — ~600-word keyword-rich prose section below the SEO FAQ. Uses `.seo-heading` / `.seo-body` classes, `var(--canvas-soft)` background, H3 subheadings. Purely static HTML — no JS.

**WhatsApp floating button** — rendered in `Layout.astro` (appears on all pages), fixed bottom-right, class `.wa-float`. Phone number is currently a placeholder (`+919876543210`) — update before going live. Styles are in the `<style is:global>` block of `Layout.astro`.

## Pricing Calculator & homepage forms (index.astro)

The pricing section and both submission forms in `src/pages/index.astro` are driven entirely by `<script is:inline>` — no framework. Key logic:

- **Class slider** (1–12): drag handle updates a fill bar and badge in real time. No CSS transition on the fill — updates synchronously via `requestAnimationFrame` to avoid lag.
- **Subjects**: Maths, Physics, Chemistry, Biology. Max 2 selectable. **Maths and Biology are hidden for class 11–12** (only available up to class 10). **Physics is shown but unclickable (disabled) for class 11–12** — displayed with note "Coming soon for 11–12"; if already selected it is deselected automatically when the slider moves to 11 or 12. Subject cards are rendered by JS into `#subject-grid`; their styles use `:global()` for this reason.
- No price is displayed — the calculator captures class + subject selection for both the "Find a Teacher" and "Batch Registration" form flows.
- **Shared state**: `currentClass` (number) and `calcSelected` (array of subject name strings) are declared as `var` in the outer `<script is:inline>` scope so all form submit handlers can read them. The IIFE that drives the calculator writes to these outer vars. Do not move them inside the IIFE.
- **Find a Teacher form** (`#find-form`): validates phone (10 digits), then `calcSelected.length > 0`, then checks `otc_user` in localStorage — opens the auth modal if not signed in, otherwise submits directly. Success replaces the form with `#form-success`.
- **Batch Registration form** (`#batch-form`): validates all `[type="tel"]` inputs (including JS-generated member fields) for 10 digits, then `calcSelected` length. Batch size (2–5) is chosen via pill buttons; member fields for members 3–5 are rendered dynamically by JS into `#batch-extra-members`. Because these elements are JS-created they need `:global()` CSS — all `.batch-member-block`, `.batch-member-row`, and `.batch-form-inner label/input` rules are `:global()`. Success shows `#batch-success`.
- **Phone validation rule** (all forms): strip non-digits and assert length === 10. Required phone fields reject empty or wrong-length numbers. Optional phone fields (modal signup, profile signup, profile edit) only validate if a value is entered — empty is allowed. Error messages appear in existing `auth-error` elements or dedicated `*-phone-error` divs adjacent to the submit button.

## Favicon

All favicon assets live in `public/` and are referenced in `Layout.astro` in this exact order (matches RealFaviconGenerator output — do not reorder):

```html
<link rel="icon" type="image/png" href="/favicon-96x96.png" sizes="96x96" />
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
<link rel="shortcut icon" href="/favicon.ico" />
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />
<meta name="apple-mobile-web-app-title" content="OTC" />
<link rel="manifest" href="/site.webmanifest" />
```

`site.webmanifest` references `web-app-manifest-192x192.png` and `web-app-manifest-512x512.png` (both maskable) for PWA/Android home-screen icons.

## Design System

UI work must follow `DESIGN.md` (Vercel-inspired design language). Key rules:

- **Colors**: Always use CSS tokens (see table in Architecture above) — never hardcode `#171717`, `#fafafa`, `#e5e5e5`, etc. in new code. Near-white `#fafafa` canvas, ink-near-black `#171717` for primary CTAs and dark bands, `#4d4d4d` for secondary text.
- **Typography**: Geist (geometric sans) at weights 400/500/600 — never 700+. **Current implementation uses Inter** (Google Fonts) as a substitute; swap to Geist when self-hosted. Headlines are sentence-case with aggressive negative letter-spacing.
- **Buttons**: 100px pill radius for marketing CTAs, 6px radius for nav-scale buttons. Black-fill primary, white-fill secondary.
- **Cards**: Stacked multi-offset shadows + 1px inset hairline border — never a single heavy drop-shadow.
- **Surfaces**: Band through `canvas-soft` (#fafafa) → `canvas` (#ffffff) → `primary` (#171717 dark band).
- **Spacing**: 4px base unit. Section padding 64–96px vertical; hero bands 192px.
- **Inner page hero**: dark `#171717` band with mono-caps eyebrow, sentence-case headline, muted subtext — see `about.astro` as reference.
- **Legal page pattern**: two-column layout with sticky sidebar TOC + prose body; sidebar hidden on mobile — see `privacy-policy.astro` as reference.
- **Interview Prep exception**: all `src/pages/interview-prep/*.astro` files use their own token set (`--ip-*` prefix) — dark `#0a0a0a` background, purple `#7c3aed` / blue `#3b82f6` / cyan `#06b6d4` gradient accent — intentionally diverging from the Vercel palette. Do not apply standard `--ink`/`--canvas` tokens there. Each sub-page duplicates the token block in its own `<style>` since there is no shared stylesheet.

## Nav

`Nav.astro` accepts `activePage?: string`. Pass the matching string from each page to highlight the correct link. Current valid values: `"about"`, `"interview-prep"`, `"contact"`, `"privacy-policy"`, `"terms"`. The "Interview Prep" nav link has an additional `.nav-link-interview` class that renders it in purple (`#7c3aed`) to distinguish it visually.

The nav contains a **theme toggle button** (`#theme-toggle`, moon/sun SVG icons) placed **outside** `.nav-links` so it remains visible on mobile alongside the hamburger. Icon visibility is controlled by CSS: `.icon-moon` hides in dark mode, `.icon-sun` hides in light mode, using `:global([data-theme="dark"]) .icon-moon { display: none }` etc. The toggle JS (`initThemeToggle`) re-initialises on `astro:after-swap`.

## interview-prep/* interactive patterns

All sub-pages share a **sticky module nav bar** (`.ip-module-nav-wrap`, `top: 64px`, `z-index: 40`) with links to all 5 modules and an `.active` class on the current page's link. Each sub-page also has prev/next links at the bottom (`.ip-pnav-back` / `.ip-pnav-next`).

Patterns by sub-page:

- **`compare.astro`** — **Flip cards**: pure CSS `rotateY(180deg)` on `.ip-flip-inner` triggered by `:hover`. Both faces use `backface-visibility: hidden`. No JS.
- **`index.astro`** — **Stat counters**: JS `IntersectionObserver` on `.ip-stats` triggers `animateCounters()` once. Each `.ip-stat-num[data-target]` counts up via `requestAnimationFrame` with cubic ease-out.
- **`questions.astro`** — **Q&A tabs**: buttons use `data-qacat` attribute; panels use `data-qapanel`. `initQATabs()` wires click handlers and toggles `.active`. Re-initialised on `astro:after-swap`. Separate from the video tab pattern below.
- **`videos.astro`** — **Video tabs**: `.ip-vtab[data-cat]` buttons toggle `.active` and show the matching `.ip-video-panel[data-panel]`. **Video modal**: clicking `.ip-video-thumb[data-videoid]` sets `#ip-video-iframe` src to `https://www.youtube-nocookie.com/embed/{id}?autoplay=1&rel=0`. Closed by button, backdrop click, or Escape. Iframe src is cleared on close to stop playback. Both re-initialised on `astro:after-swap`.
- **`plan.astro`** — **Checklist**: 12 items, each tagged with a category (`research`, `content`, `practice`, `logistics`). Checkboxes persist per-item to `localStorage` under key `ip_check_{id}`. Progress bar width set inline. Completion state shows `#ip-checklist-done`. Reset button clears all localStorage keys and re-runs `updateProgress()`. Re-initialised on `astro:after-swap`.
