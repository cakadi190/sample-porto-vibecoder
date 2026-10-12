# coba-cv: a personal CV page you can make yours

A sample project for a one-person CV / résumé site, built on **Nuxt 4 (stable)**.
Clone it, add your details, then make it look like nobody else's.

> 📄 **Start with the product spec: [`docs/PRD.md`](docs/PRD.md).**
> It covers the goals, the data contract, and which parts are open to creative changes.

## Quick start

Requirements: **Node.js 24 LTS** (see `.nvmrc`) and **Bun**. Choosing an OS,
WSL2, Docker, or Dev Containers? See [`docs/DEVELOPER_ENVIRONMENT.md`](docs/DEVELOPER_ENVIRONMENT.md).

```bash
bun install
bun run dev        # http://localhost:3000
```

## Make it yours

1. **Your content:** edit `app/data/cv.ts`. All the text on the page comes from this file.
2. **Your look:** change the design tokens in `app/assets/css/` and the components in `app/components/cv/`.
   Layout, style, and motion are all yours to change (see PRD §8 for ideas).
3. **Ship it:** `bun run generate`, then deploy `.output/public` to any static host.

## Working with AI agents

This repo includes instructions that tell coding agents to read the PRD before doing anything:

| Agent              | Entry file                         |
| ------------------ | ---------------------------------- |
| All (shared rules) | `AGENTS.md`                        |
| Claude Code        | `CLAUDE.md`                        |
| Gemini CLI         | `GEMINI.md`                        |
| GitHub Copilot     | `.github/copilot-instructions.md`  |
| Cursor             | `.cursor/rules/00-read-prd.mdc`    |

Try a prompt like *"Read the PRD, then redesign my CV as a retro terminal"*.

## Scripts

| Command             | What it does                    |
| ------------------- | ------------------------------- |
| `bun run dev`       | Dev server                      |
| `bun run build`     | Production build (SSR)          |
| `bun run generate`  | Static site → `.output/public`  |
| `bun run preview`   | Preview the production build    |
