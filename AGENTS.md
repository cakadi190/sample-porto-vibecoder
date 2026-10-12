# AGENTS.md

Instructions for every AI coding agent working in this repository (Claude Code,
Codex, Cursor, Copilot, Gemini, Windsurf, Aider, etc.).

## 0. Read the PRD first, every time

**Before you plan, answer, or edit anything, read [`docs/PRD.md`](docs/PRD.md) in full.**
It defines the product (a personal CV page sample project), the data contract,
and which parts are creative versus fixed. If a request conflicts with the PRD,
do what the user asks, but tell them about the conflict first.

## 1. Project at a glance

- Personal CV / résumé site, meant to be cloned and remixed.
- **Nuxt 4 stable** (`^4.6.x`), Vue 3, TypeScript, `app/` as srcDir.
  Do not upgrade to alpha/rc/nightly releases or enable `future.compatibilityVersion: 5`.
- Package manager: **Bun** (`bun install`, `bun run dev`, `bun run generate`). Don't mix in npm/pnpm/yarn lockfiles.
- Runtime: **Node.js 24 LTS** (`.nvmrc`; `engines` mirrors Nuxt's range). Don't add Docker,
  Dev Containers, or VM config unless asked. See [`docs/DEVELOPER_ENVIRONMENT.md`](docs/DEVELOPER_ENVIRONMENT.md).

## 2. Hard rules (the contract)

1. All CV content lives in `app/data/cv.ts`, typed by `app/types/cv.ts`.
   Never hard-code names, jobs, or other CV text in components.
2. Components get data through `useCv()` (`app/composables/useCv.ts`).
3. The `CV` type may be **extended** but not renamed or narrowed.
4. Optional sections with no data render nothing.
5. `/` must render the CV, and `bun run generate` must keep producing a working static site.
6. Keep the print stylesheet, accessibility baseline (semantic HTML, AA contrast,
   reduced motion), and `useSeoMeta` + JSON-LD intact.
7. Design tokens (colors, fonts, spacing) stay centralized as CSS custom properties.
8. Sample data must be clearly fictional. Never commit real personal data unless
   the repository owner explicitly asks.

## 3. Be creative inside the creative layer

Layout, visual style, typography, motion, extra pages, and interactive touches
are all open (see PRD §8). When a user asks for a "creative" or "unique" look,
commit to a bold, coherent concept instead of a generic template, as long as
the hard rules above still hold.

## 4. Conventions

- `<script setup lang="ts">` only, Composition API.
- Use Nuxt auto-imports. Don't manually import auto-imported composables or components.
- CV section components go in `app/components/cv/` (auto-named `<CvHero/>`, `<CvExperience/>`, …).
- Keep dependencies light, and explain any new one in the commit message.

## 5. Definition of done

Before reporting a task as finished, run:

```bash
bunx nuxi typecheck
bun run generate
```

Then check the change against the acceptance criteria in PRD §9.
