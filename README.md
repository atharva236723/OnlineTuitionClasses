![Online Tuition Classes](public/og-poster.svg)

# Online Tuition Classes

**Online Tuition Classes** is a platform that connects students from **Class 1 to Class 12** with verified, experienced tutors for personalised one-on-one or batch online tuition.

Parents and students can browse tutors by subject and class, register for batch sessions at affordable pricing, and get matched with the right teacher — all from a single website. No app download needed. Sessions are conducted online with flexible timings to suit school schedules.

## What We Offer

- **Find a Teacher** — submit your class and subject, get matched with a verified tutor
- **Batch Registration** — register a group of 2–5 students for discounted batch sessions
- **Pricing Calculator** — instantly see session pricing based on class and subjects selected
- **Google Sign-In** — quick and secure login with your Google account

## Subjects

| Subject | Available for |
|---|---|
| Maths | Class 1 – 12 |
| Physics | Class 1 – 10 |
| Chemistry | Class 1 – 12 |
| Biology | Class 1 – 10 |

## Pages

| URL | Description |
|---|---|
| `/` | Home — hero, how it works, pricing calculator, find a teacher form, batch registration |
| `/about` | Our mission and why we started Online Tuition Classes |
| `/contact` | Send us a message |
| `/profile` | Sign up, log in, or edit your profile |
| `/privacy-policy` | How we handle your data |
| `/terms` | Terms and conditions for using our platform |

## Tech Stack

| | |
|---|---|
| Framework | [Astro 5](https://astro.build) — static site with SPA-style navigation |
| Styling | Scoped CSS with custom properties (no external CSS framework) |
| Smooth Scroll | [Lenis](https://github.com/darkroomengineering/lenis) |
| Auth | Frontend-only via `localStorage` + Google Identity Services |
| Forms | [Formspree](https://formspree.io) — submissions go to email |
| Hosting | Static deployment (no backend or database) |

## Local Development

```sh
npm install       # install dependencies
npm run dev       # start dev server at localhost:4321
npm run build     # build for production
npm run preview   # preview production build locally
```
