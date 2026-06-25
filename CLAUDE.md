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
| `src/pages/index.astro` | `/` | Home — hero, How It Works, Pricing, Find a Teacher form |
| `src/pages/about.astro` | `/about` | Mission, verification process, stats |
| `src/pages/contact.astro` | `/contact` | Contact info + message form |
| `src/pages/privacy-policy.astro` | `/privacy-policy` | Privacy policy (7 sections, sticky TOC) |
| `src/pages/terms.astro` | `/terms` | Terms & conditions (9 sections, sticky TOC) |

## Architecture

This is an [Astro](https://docs.astro.build) project using TypeScript (strict mode).

- `src/pages/` — file-based routing; each `.astro` file is a URL
- `src/layouts/Layout.astro` — HTML shell: `<head>`, global resets, Inter font, and the **Lenis smooth-scroll bootstrap** (`<script>` at the bottom of `<body>`)
- `src/components/Nav.astro` — shared sticky nav used on every page; accepts `activePage?: string` prop to highlight the current link. Links: About, Contact, Privacy, Terms (Privacy + Terms hidden on mobile ≤600px), plus the "Find a Teacher" CTA.
- `src/components/Footer.astro` — shared dark footer with Company / Legal / Students link columns; used on every page
- `src/assets/` — images processed by Astro's asset pipeline
- `public/` — static files served as-is (favicons, etc.)

Pages import `Layout`, `Nav`, and `Footer`, then compose their content between them. The frontmatter (`---` fences) runs server-side at build time.

**Smooth scrolling** is handled by [Lenis](https://github.com/darkroomengineering/lenis) (`lenis` npm package, `autoRaf: true`), initialised once in `Layout.astro`. It intercepts `a[href^="#"]` anchor clicks so in-page hash navigation stays smooth. Do not add `scroll-behavior: smooth` to new CSS — Lenis already handles it.

**Styles** are colocated as `<style>` blocks inside each `.astro` file using CSS custom properties (`--ink`, `--canvas`, etc.). There is no global stylesheet beyond the reset in `Layout.astro`. Token definitions are repeated per-page because Astro scopes component styles.

## Design System

UI work must follow `DESIGN.md` (Vercel-inspired design language). Key rules:

- **Colors**: Near-white `#fafafa` canvas, ink-near-black `#171717` for primary CTAs and dark bands, `#4d4d4d` for secondary text. The multi-stop mesh gradient (cyan → blue → magenta → amber) is the only decorative element — hero scale only, never miniaturised.
- **Typography**: Geist (geometric sans) for all display/body/buttons at weights 400/500/600 — never 700+. Geist Mono for code blocks and technical labels. Headlines are sentence-case with aggressive negative letter-spacing, often period-terminated. **Current implementation uses Inter as an open-source substitute** (loaded from Google Fonts); swap to Geist when a self-hosted copy is available.
- **Buttons**: 100px pill radius for marketing CTAs (`button-primary`/`button-secondary`), 6px radius for nav-scale buttons. Black-fill primary, white-fill secondary.
- **Cards**: Stacked multi-offset shadows + 1px inset hairline border — never a single heavy drop-shadow.
- **Surfaces**: Cycle bands through `canvas-soft` (#fafafa) → `canvas` (#ffffff) → polarity-flipped `primary` (#171717 dark band).
- **Spacing**: 4px base unit. Section padding `64–96px` vertical; hero bands `192px`.
- **Page hero pattern** (inner pages): dark `#171717` band with mono-caps eyebrow, large sentence-case headline, muted subtext — see `about.astro` or `contact.astro` as the reference.
- **Legal page pattern**: two-column layout with sticky sidebar TOC + prose body; sidebar hidden on mobile — see `privacy-policy.astro` or `terms.astro`.
