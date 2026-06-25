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

**Online Tuition Classes** — a frontend-only marketing site for an online tutoring service matching students (Class 1–12) with verified teachers. No backend, database, or payment integration yet; form submissions are handled client-side with a success-state toggle.

This is a **multi-page application (MPA)**. Pages and their URLs:

| File | URL | Purpose |
|---|---|---|
| `src/pages/index.astro` | `/` | Home — hero, How It Works, Pricing calculator, Find a Teacher form |
| `src/pages/about.astro` | `/about` | Mission, verification process, stats |
| `src/pages/contact.astro` | `/contact` | Contact info + message form |
| `src/pages/privacy-policy.astro` | `/privacy-policy` | Privacy policy (7 sections, sticky TOC) |
| `src/pages/terms.astro` | `/terms` | Terms & conditions (9 sections, sticky TOC) |

## Architecture

This is an [Astro](https://docs.astro.build) project using TypeScript (strict mode).

- `src/pages/` — file-based routing; each `.astro` file is a URL
- `src/layouts/Layout.astro` — HTML shell: `<head>`, global resets, Inter font, and the **Lenis smooth-scroll bootstrap** (`<script>` at the bottom of `<body>`)
- `src/components/Nav.astro` — shared sticky nav; accepts `activePage?: string` prop. Has a circular icon-only profile button (no text). Privacy + Terms links hidden on mobile ≤600px.
- `src/components/Footer.astro` — shared dark footer with Company / Legal / Students link columns
- `src/assets/` — images processed by Astro's asset pipeline
- `public/` — static files served as-is

Pages import `Layout`, `Nav`, and `Footer`, then compose their content between them. Frontmatter (`---` fences) runs server-side at build time.

**Smooth scrolling** is handled by [Lenis](https://github.com/darkroomengineering/lenis) (`lenis` npm package, `autoRaf: true`), initialised once in `Layout.astro`. Do not add `scroll-behavior: smooth` to CSS — Lenis handles it.

**Styles** are colocated `<style>` blocks inside each `.astro` file using CSS custom properties (`--ink`, `--canvas`, etc.). There is no global stylesheet beyond the reset in `Layout.astro`. Token definitions are repeated per-page because Astro scopes component styles.

**Astro CSS scoping gotcha**: styles in `<style>` blocks are scoped by adding a `data-astro-cid-*` attribute to elements in the template. Elements created dynamically via JavaScript (`innerHTML`, `createElement`) do NOT receive this attribute, so scoped selectors won't match them. Use `:global(.classname)` for any styles targeting JS-rendered elements.

## Pricing Calculator (homepage)

The interactive pricing section in `src/pages/index.astro` is driven entirely by `<script is:inline>` — no framework. Key logic:

- **Class slider** (1–12): drag handle updates a fill bar and badge in real time. No CSS transition on the fill — updates synchronously via `requestAnimationFrame` to avoid lag.
- **Subjects**: Maths, Physics, Chemistry, Biology. Max 2 selectable. **Maths and Biology are hidden for class 11–12** (only available up to class 10). Subject cards are rendered by JS into `#subject-grid`; their styles use `:global()` for this reason.
- No price is displayed — the calculator currently only captures class + subject selection for the "Find a Teacher" form flow.

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
