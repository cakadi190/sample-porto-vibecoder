# PRD — Personal CV Page (Nuxt Sample Project)

| Field            | Value                                             |
| ---------------- | ------------------------------------------------- |
| Status           | Active                                            |
| Version          | 1.0                                               |
| Last updated     | 2026-10-12                                        |
| Framework target | **Nuxt 4.x stable** (currently `4.6.1`), Vue 3.5  |
| Package manager  | Bun only (`bun.lock` is committed). A fork may switch to pnpm, but must use exactly one lockfile |
| Runtime          | Node.js 24 LTS (≥ 24.15.0) or 22 (≥ 22.22.3), see [`DEVELOPER_ENVIRONMENT.md`](DEVELOPER_ENVIRONMENT.md) |

> **For AI agents:** This document is the source of truth for this repository.
> Read it fully before planning or writing code. If a user request conflicts
> with this PRD, follow the user but point out the conflict.

---

## 1. Summary

`coba-cv` is a **sample project**: a one-person CV / résumé website built with
Nuxt. It is meant to be **cloned and remixed**. Each person who clones it should
be able to:

1. Swap in their own CV data by editing **one data file**, and
2. Restyle the page in any way they like (minimal, brutalist, terminal-themed,
   magazine-style, 3D, whatever) **without breaking the data contract**.

The repo ships a clean, working baseline. Creative choices belong to whoever
clones it.

## 2. Goals

| #  | Goal                                                                            |
| -- | ------------------------------------------------------------------------------- |
| G1 | A working, good-looking CV page right after `bun install && bun run dev`.       |
| G2 | All personal content lives in **one typed data file**. No CV text hard-coded in components. |
| G3 | Clear split between the **stable contract** (data shape, routes, SEO) and the **creative layer** (layout, styling, motion). |
| G4 | Deploys as a **static site** (`nuxt generate`) to any static host.              |
| G5 | Easy for humans and AI agents to extend: small components, typed data, documented conventions. |

## 3. Non-goals

- No CMS, database, auth, or admin panel.
- No backend/API routes beyond what Nuxt needs for static generation.
- No multi-user or multi-tenant support. One site = one person.
- No pinned design system. The baseline styling is a starting point only.

## 4. Target users

| Persona             | Need                                                                        |
| ------------------- | --------------------------------------------------------------------------- |
| **Cloner / owner**  | Wants a personal CV site fast, then wants to make it look like *theirs*.    |
| **Learner**         | Uses the repo to learn Nuxt 4 conventions (`app/` dir, auto-imports, `useSeoMeta`). |
| **AI coding agent** | Gets asked "make my CV look like X" or "add my projects". Needs clear rules on what it may change. |
| **Visitor**         | Recruiter or peer reading the CV on desktop, phone, or paper (print).       |

## 5. Functional requirements

### 5.1 Content sections

The CV page **must** render these sections when the data for them exists:

| Section       | Required | Notes                                                    |
| ------------- | -------- | -------------------------------------------------------- |
| Profile/Hero  | Yes      | Name, headline/title, location, short summary, avatar (optional). |
| Contact/links | Yes      | Email, website, socials (GitHub, LinkedIn, etc.).        |
| Experience    | Yes      | Role, company, period, location, highlights (bullets).   |
| Education     | Yes      | Institution, degree, period, notes.                      |
| Skills        | Yes      | Grouped (e.g. "Languages", "Frameworks", "Tools").       |
| Projects      | Optional | Name, description, tech, link.                           |
| Certifications| Optional | Name, issuer, date, link.                                |
| Languages     | Optional | Spoken languages + proficiency.                          |
| Custom        | Optional | Free-form extra sections (awards, talks, volunteering…). |

Rules:
- If an optional section has no data, **hide it completely**. No empty headings.
- Section **order** is controlled by the data file, not hard-coded.

### 5.2 Single source of data — the data contract

- Location: `app/data/cv.ts`
- Types: `app/types/cv.ts`
- The data file exports a default object typed as `CV`.
- Components read data **only** through a composable: `useCv()` in
  `app/composables/useCv.ts`.

Minimum type shape (you may extend it, but don't rename or remove fields):

```ts
// app/types/cv.ts
export interface CvLink { label: string; url: string; icon?: string }

export interface CvProfile {
  name: string
  headline: string
  location?: string
  summary: string
  avatar?: string          // path under /public or absolute URL
  email?: string
  links: CvLink[]
}

export interface CvExperience {
  role: string
  company: string
  location?: string
  start: string            // ISO-ish: "2023-04" or "2023"
  end?: string             // undefined = "Present"
  highlights: string[]
  tech?: string[]
}

export interface CvEducation {
  institution: string
  degree: string
  start?: string
  end?: string
  notes?: string[]
}

export interface CvSkillGroup { name: string; items: string[] }

export interface CvProject {
  name: string
  description: string
  url?: string
  tech?: string[]
}

export interface CvCertification { name: string; issuer: string; date?: string; url?: string }

export interface CvSpokenLanguage { name: string; level: string }

export interface CvCustomSection { id: string; title: string; items: string[] }

export type CvSectionId =
  | 'experience' | 'education' | 'skills' | 'projects'
  | 'certifications' | 'languages' | `custom:${string}`

export interface CV {
  profile: CvProfile
  sections: CvSectionId[]  // render order
  experience: CvExperience[]
  education: CvEducation[]
  skills: CvSkillGroup[]
  projects?: CvProject[]
  certifications?: CvCertification[]
  languages?: CvSpokenLanguage[]
  custom?: CvCustomSection[]
  meta?: { siteUrl?: string; ogImage?: string; locale?: string }
}
```

The baseline ships with **clearly fictional placeholder data** (e.g.
"Alex Example"). Never commit real personal data as the sample.

### 5.3 Pages and routes

| Route | Purpose                                   | Required |
| ----- | ----------------------------------------- | -------- |
| `/`   | The CV page itself                        | Yes      |
| Any other | Cloners may add (`/projects/[slug]`, `/blog`, …) | Optional |

The repo uses Nuxt's `app/pages/` routing. `app/app.vue` must contain
`<NuxtPage />` (and keep `<NuxtRouteAnnouncer />`).

### 5.4 SEO and sharing

- Use `useSeoMeta()` for title, description, `og:*`, and `twitter:*`, all
  derived from `cv.profile` / `cv.meta`.
- Set `<html lang>` from `cv.meta.locale` (default `en`).
- Add JSON-LD `Person` structured data built from the profile.

### 5.5 Print / PDF

- The page **must** print well (`Ctrl/Cmd+P` → "Save as PDF") using
  `@media print` styles: no nav, no animations, readable fonts, no cut-off
  sections, links shown as text where useful.
- A visible "Download / Print" control is recommended (it can just call
  `window.print()`).

### 5.6 Theming

- Support light and dark mode, following `prefers-color-scheme` by default.
- All colors, fonts, spacing, and radii come from **CSS custom properties**
  (design tokens) in one place (e.g. `app/assets/css/tokens.css`). Swapping the
  look of the site should mostly mean editing tokens and components, never data.

## 6. Non-functional requirements

| Area           | Requirement                                                              |
| -------------- | ------------------------------------------------------------------------ |
| Performance    | Lighthouse ≥ 90 for Performance, Accessibility, Best Practices, SEO on the generated site. |
| Accessibility  | Semantic landmarks (`header`, `main`, `section` with headings), WCAG AA contrast, visible focus, alt text, respects `prefers-reduced-motion`. |
| Responsiveness | Works from 320px up to wide desktop. No horizontal scroll.               |
| Static output  | `bun run generate` succeeds and the output in `.output/public` works with no server. |
| Type safety    | TypeScript throughout. `nuxi typecheck` passes.                          |
| Dependencies   | Keep them light. Every new dependency should be justified in the PR/commit message. |

## 7. Technical constraints

- **Nuxt 4 stable** (`^4.6.x`). Use the Nuxt 4 directory layout (`app/` as
  `srcDir`). Do **not** enable `future.compatibilityVersion: 5` or use
  alpha/rc/nightly Nuxt builds.
- Vue 3 Composition API with `<script setup lang="ts">` only.
- Rely on Nuxt auto-imports (components, composables, utils); don't add
  redundant manual imports for them.
- Styling approach is **open** (plain CSS, Tailwind, UnoCSS, Nuxt UI, …), but
  design tokens must stay centralized (see 5.6).
- Official Nuxt modules are welcome when they earn their place (e.g.
  `@nuxt/image`, `@nuxt/fonts`, `@nuxtjs/color-mode`, `@nuxt/icon`).
- **Runtime & tooling:** Node.js within `nuxt@4.6.x` `engines`
  (`^22.22.3 || ^24.15.0 || >=26.0.0`; Node 24 LTS recommended), Bun as the only
  package manager. Native development is the default. Docker, Dev Containers,
  and VMs are optional. Full details: [`docs/DEVELOPER_ENVIRONMENT.md`](DEVELOPER_ENVIRONMENT.md).
- **Deployment:** static (`nuxt generate`) is the default. A Node server build
  (`node .output/server/index.mjs`) on a Linux VPS is also supported.

### Suggested structure

```
app/
  app.vue                  # <NuxtRouteAnnouncer/> + <NuxtLayout><NuxtPage/></NuxtLayout>
  pages/index.vue          # the CV page: composes sections in cv.sections order
  layouts/default.vue
  components/
    cv/                    # CvHero, CvExperience, CvSkills, CvSection … (auto-imported as <CvHero/> etc.)
  composables/useCv.ts
  data/cv.ts               # ← THE file cloners edit
  types/cv.ts
  assets/css/tokens.css    # ← design tokens
  assets/css/main.css
public/                    # avatar, og image, favicon
docs/PRD.md                # this document
docs/DEVELOPER_ENVIRONMENT.md  # runtime, OS & tooling recommendations
```

## 8. Creative freedom: what you may and may not change

This project exists so people can get creative. The table below sets the
boundaries.

| 🎨 Free to change (creative layer)                   | 🔒 Keep stable (contract)                                  |
| ---------------------------------------------------- | ---------------------------------------------------------- |
| Layout: single column, sidebar, bento grid, timeline, slides, terminal UI… | The `CV` type: add fields, but don't rename or remove them. |
| Visual style, typography, color palette, tokens      | All CV content stays in `app/data/cv.ts`                    |
| Motion, transitions, scroll effects (respect reduced motion) | Components read data through `useCv()`              |
| Extra pages, easter eggs, interactive widgets        | `/` renders the CV; static `generate` keeps working        |
| Styling library choice                               | Print stylesheet, a11y baseline, SEO meta                  |
| Section component markup                             | Empty optional sections stay hidden                        |

**Idea starters for cloners:** terminal/CLI-style CV, newspaper front page,
retro OS window, Swiss/brutalist typography, bento grid, scroll-driven
timeline, game-like "character sheet", multi-language toggle (i18n),
auto-generated OG image.

## 9. Acceptance criteria (baseline)

- [ ] `bun install && bun run dev` serves the CV at `http://localhost:3000`.
- [ ] Editing only `app/data/cv.ts` changes every piece of displayed content.
- [ ] Reordering `cv.sections` reorders the page.
- [ ] Removing `projects` from the data hides the Projects section with no layout gaps.
- [ ] Light and dark mode both render with AA contrast.
- [ ] Print preview gives a clean 1–2 page document.
- [ ] `bun run generate` succeeds, and the output works when opened from a static server.
- [ ] `bunx nuxi typecheck` passes.
- [ ] No real personal data in the committed sample.

## 10. Milestones

| Milestone | Scope                                                                 |
| --------- | --------------------------------------------------------------------- |
| M1        | Types, placeholder data, `useCv()`, `/` page with all required sections, plain token-based CSS. |
| M2        | Dark mode, print stylesheet, SEO + JSON-LD, accessibility pass.       |
| M3        | Static generation verified, README "Make it yours" guide, Lighthouse check. |
| M4+       | Open-ended: the cloner's creative direction.                          |

## 11. Open questions

- Should the baseline ship a styling library (e.g. Tailwind) or stay with plain CSS? *Default: plain CSS + tokens, to keep the baseline dependency-free.*
- Should i18n (multiple CV languages) be part of the baseline? *Default: no, but the data shape should not block it.*
