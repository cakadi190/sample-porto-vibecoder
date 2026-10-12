# Developer Environment — Compatibility & Recommendations

| Field        | Value                                                        |
| ------------ | ------------------------------------------------------------ |
| Applies to   | `coba-cv`, Nuxt **4.6.x** (verified against `nuxt@4.6.1`)   |
| Parent spec  | [`docs/PRD.md`](PRD.md) §7 Technical constraints              |
| Last checked | 2026-10-12                                                   |

> **TL;DR:** Use Git, **Node.js 24 LTS**, and **Bun** as the package manager,
> running natively on your OS. On Windows, use WSL2 only if native HMR feels
> slow. Docker, Dev Containers, and VMs are **optional** and not needed to work
> on a static CV site.

---

## 1. Runtime and package manager (mandatory)

### 1.1 Node.js

| Source | Requirement |
| ------ | ----------- |
| Nuxt docs (installation) | "Node.js 22.x or newer (but we recommend the active LTS release)"; even-numbered versions preferred |
| `nuxt@4.6.1` `package.json` → `engines.node` | `^22.22.3 \|\| ^24.15.0 \|\| >=26.0.0` |
| Vite 8 / Nitro 2.13 (`engines.node`) | `^20.19.0 \|\| >=22.12.0` (looser than Nuxt, so Nuxt's range is what binds) |

**Decision: Node.js 24 LTS (≥ 24.15.0).** It is an LTS line inside Nuxt's
supported range. Node 22 ≥ 22.22.3 also works. Avoid odd-numbered and
"Current" releases for daily work. Note that the package's `engines` minimums
are **stricter than the docs**: an older Node 22.x patch release is *outside*
the supported range.

The repo pins this in two places:
- `.nvmrc` → `24` (read by `nvm`, `fnm`, and `volta`-style tools)
- `package.json` → `engines.node` mirrors Nuxt's range

Install Node with a version manager, not a system package (`apt` versions are
often too old):

| OS | Recommended installer |
| -- | --------------------- |
| Linux / WSL2 / macOS | [`fnm`](https://github.com/Schniz/fnm) or [`nvm`](https://github.com/nvm-sh/nvm) |
| Windows (native) | `fnm` (`winget install Schniz.fnm`) or the official Node.js LTS installer |

### 1.2 Package manager: Bun

**Decision: Bun ≥ 1.1, used only as the package manager and script runner.**

Why Bun:
- `bun.lock` is already committed, and the PRD names Bun. Using one package
  manager consistently prevents lockfile drift and "works on my machine" bugs.
- Dependency installs are fast and use a global cache, which helps most on slow
  disks and low-spec machines.
- Official binaries exist for **Windows x64/ARM64 (10 1809+), macOS x64/ARM64
  (13.0+), and Linux x64/ARM64 (glibc ≥ 2.17 or musl)**. On x64, the CPU must
  support SSE4.2 (any Intel Core since Nehalem, AMD since Bulldozer).

Important detail: `bun run dev` runs the `nuxt` CLI on **Node.js**. Bun is not
replacing the runtime here, so Node is still mandatory. Don't pass `--bun`
unless you are deliberately testing the Bun runtime.

Fallbacks: if Bun can't be installed (for example, an old x64 CPU without
SSE4.2), use **pnpm**. Keep that fork consistent: delete `bun.lock`, commit
`pnpm-lock.yaml`, and never mix two lockfiles.

### 1.3 Editor and Git (mandatory / recommended)

- **Git**: mandatory.
- **Editor**: Nuxt has no IDE requirement. It recommends **VS Code + the
  official Vue extension (Vue - Official)** or WebStorm. On 8 GB machines,
  VS Code or a terminal editor (Neovim + Vue language tools) is lighter than a
  full JetBrains IDE.

---

## 2. Three different things called "cloning"

These solve different problems and are **not interchangeable**:

| Layer | What it copies | Tool for this project | Needed? |
| ----- | -------------- | --------------------- | ------- |
| **1. Repository cloning** | Source code + history | `git clone` (or GitHub "Use this template" / fork) | **Mandatory** |
| **2. Dev-environment cloning** | Runtime versions, dependencies, editor config | `.nvmrc` + `engines` + `bun.lock` (lightweight). Optionally a Dev Container | Lightweight pinning: **mandatory**. Container: optional |
| **3. OS / machine virtualization** | A whole separate OS or kernel | WSL2 (Windows), Docker (Linux kernel in a VM on macOS/Windows), VirtualBox/Hyper-V/UTM | **Not required**. WSL2 is an optional performance workaround on Windows |

For a single Nuxt site with no database or external services, layer 2 is
mostly covered by **version pinning + a lockfile**. Containers add
reproducibility only when a team needs byte-identical toolchains.

---

## 3. Tool decision matrix

| Tool | Solves | Overhead | Value for this project | Verdict |
| ---- | ------ | -------- | ---------------------- | ------- |
| **Native Node.js + Git** | Running and building the app | Lowest: no VM, direct filesystem | Covers 100% of the PRD's needs | ✅ **Default** |
| **Version manager (fnm/nvm) + `.nvmrc`** | Matching the Node version | Negligible | Prevents `engines` mismatches | ✅ **Recommended** |
| **WSL2** (Windows) | Linux toolchain on Windows; fixes slow Windows file watching/HMR, as Nuxt's docs suggest | A Linux VM, by default allowed up to **50% of host RAM** | High when native Windows HMR is slow, otherwise neutral | 🟡 Optional (recommended on Windows with ≥16 GB) |
| **Dev Containers** (VS Code / JetBrains / Codespaces) | Identical toolchain for a team, one-click onboarding | Needs a container engine (a VM on Windows/macOS) | Useful for teams/workshops, not for solo use | 🟡 Optional |
| **Docker / Compose** | Isolating services; production-like Node server testing | Container engine + image builds | Low: this project has no DB or services | 🟡 Optional, only for deployment rehearsal |
| **Full VM** (VirtualBox, Hyper-V, UTM, Parallels) | Testing a different OS | Highest: a full guest OS, its own RAM/disk | None for normal development | ❌ Not recommended (only to reproduce an OS-specific bug or rehearse a VPS setup) |
| **Cloud dev env** (Codespaces, Gitpod-style) | Zero local setup | Network latency, usage quota | Handy on locked-down or very weak machines | 🟡 Optional |

---

## 4. Hardware scenarios

These are **evaluation scenarios, not minimum requirements**. "Comfort" assumes
the whole working set is open at once: an editor, `bun run dev`, and a browser
with DevTools. No exact RAM/CPU numbers are claimed here because they haven't
been measured for this repo. The bottleneck column describes how each setup
tends to behave, qualitatively.

| # | Host OS | RAM | CPU | Recommended workflow | Isolation tool | Docker / VM needed? | Comfort | Main bottlenecks → mitigations | Suitable for |
| - | ------- | --: | --- | -------------------- | -------------- | ------------------- | ------- | ------------------------------ | ------------ |
| W1 | Windows | 8 GB | 2C/2T | **Native Windows** Node 24 + Bun, project on an NTFS SSD, open via `http://127.0.0.1:3000` | None | No. Avoid both | **Constrained** | RAM pressure from editor + browser + dev server → a single browser window, Nuxt DevTools off, close background apps. Slow file watching → exclude the project folder from real-time antivirus scanning *only if your policy allows it*. WSL2 adds a VM, so try it only if HMR is unusable, and cap it with `.wslconfig` (`memory=3GB`, `processors=2`) | Local dev; build/generate checks |
| W2 | Windows | 16 GB | 4C/8T | **WSL2 (Ubuntu)**, project in the Linux FS (`~/projects`), VS Code + WSL extension. Native Windows is also fine | WSL2 | No Docker needed | **Comfortable** | Cross-filesystem I/O when the repo sits on `/mnt/c` → keep it in `~/`. WSL2 memory growth → optional `.wslconfig` cap (e.g. 6–8 GB) | Local dev + testing |
| W3 | Windows | 32 GB | 8C/16T | **WSL2** as the daily driver. Docker Desktop (WSL2 backend) and Dev Containers available when needed | WSL2; Docker / Dev Containers optional | Optional | **Comfortable → excessive** for this project alone | Few. Just don't leave idle containers running | Full dev, testing, deployment rehearsal |
| L1 | Linux | 8 GB | 2C/2T | **Native** Node 24 (via fnm) + Bun + Git | None | No | **Acceptable** | RAM when running a heavy IDE → use a light editor or terminal. Watcher limits → raise `fs.inotify.max_user_watches` only if you see `ENOSPC` | Local dev; generate checks |
| L2 | Linux | 16 GB | 4C/8T | **Native**. This is the reference setup | None | Docker only to test a containerized deploy | **Comfortable** | None significant | Dev + testing |
| L3 | Linux | 32 GB | 8C/16T | **Native** for dev. Docker/Dev Containers for team parity or deploy rehearsal | Docker / Dev Containers optional | Optional | **Excessive** for this project's scope | Over-engineering → still default to native | Dev, integration and deployment testing |
| M1 | macOS | 8 GB | 4 cores or equivalent (Apple Silicon or Intel) | **Native** Node 24 + Bun via fnm/Homebrew | None | No. Docker on macOS runs a Linux VM | **Acceptable** (Apple Silicon) / **Constrained** (older Intel) | Memory pressure and swap with many tabs → one browser, DevTools off when not needed | Local dev; generate checks |
| M2 | macOS | 16 GB | 8 cores or equivalent | **Native**. Containers only on demand (Docker Desktop, OrbStack, Colima) | Optional | Optional | **Comfortable** | Containers: don't bind-mount host `node_modules` into a Linux container (different native binaries) | Dev + testing |

### Architecture notes

- **x64 and ARM64 are both first-class**: Node.js and Bun ship native builds for
  Windows, macOS, and Linux on both. Apple Silicon and Windows-on-ARM need no
  emulation.
- The Nuxt 4 toolchain (Vite 8 and others) pulls **platform-specific native
  binaries** through optional dependencies. This means:
  - Always run `bun install` **inside** the environment that will run the code.
    Never copy `node_modules` between Windows ↔ WSL2, host ↔ container, or
    x64 ↔ ARM64.
  - In Docker/Dev Containers on macOS or Windows, keep `node_modules` in a
    container volume, not on the bind-mounted host folder.
- **Old x64 CPUs without SSE4.2** can't run Bun. Use pnpm or npm instead.

---

## 5. Comfort and resource efficiency

What actually costs resources while working on this project, roughly from
largest to smallest:

1. **The browser + DevTools.** Usually the biggest consumer. Keep one window for the CV page.
2. **The IDE + Vue/TypeScript language server.** VS Code is lighter than JetBrains on 8 GB.
3. **`nuxt dev` (Vite + Nitro + Nuxt DevTools).** Cold start is the slowest part. HMR after that is quick when the filesystem is fast.
4. **Virtualization layers** (WSL2 VM, Docker VM on macOS/Windows). Each adds a guest kernel and memory that's reserved or cached.
5. **The package manager.** Only matters during installs. Bun's global cache makes reinstalls cheap.

Practical optimizations, especially for 8 GB / 2-core machines:

- **Use one package manager** (Bun) and one lockfile. Never mix `npm install` into a Bun checkout.
- **Keep files next to the toolchain**: Windows tools → NTFS path; WSL2 tools → `~/` inside Linux (Microsoft explicitly advises against `/mnt/c` for performance).
- **On Windows, open `http://127.0.0.1:3000` instead of `localhost`.** Nuxt's docs report faster loads that way.
- **Disable Nuxt DevTools when not needed**: `devtools: { enabled: false }` in `nuxt.config.ts`, or toggle it per machine.
- **Run only what the task needs**: no idle containers, no extra dev servers, no Storybook-style side tools for this scope.
- **Use static generation.** The CV is mostly static, so `bun run generate` + a static preview checks the real output without keeping a dev server running.
- **Cap WSL2 when RAM is tight** using `%UserProfile%\.wslconfig`:

  ```ini
  [wsl2]
  memory=4GB      # default is 50% of host RAM
  processors=2    # default is all logical processors
  ```

  Then run `wsl --shutdown` and reopen.

---

## 6. Recommendations

### 6.1 Ranked picks

| Category | 1st choice | 2nd choice | Why |
| -------- | ---------- | ---------- | --- |
| **Best overall** | Native Node 24 + Bun + Git on Linux or macOS | WSL2 on Windows | No virtualization layer, direct filesystem, fastest HMR, simplest debugging |
| **Windows developer** | WSL2 + VS Code WSL extension, repo in `~/` (≥ 16 GB) | Native Windows (≤ 8 GB, or if WSL2 is unavailable) | WSL2 matches the Linux VPS target and avoids Windows file-watching slowness. Native wins when RAM is scarce |
| **Linux developer** | Native | — | Already matches the deploy target. Containers add nothing for daily dev |
| **Low-spec hardware** | Native on the host OS, light editor, DevTools off | Cloud dev env (Codespaces-style) | Every extra layer competes for the same 8 GB |
| **Team needing reproducibility** | `.nvmrc` + `engines` + `bun.lock` (already in repo) | Dev Container (see §8.2) | Pinning covers most drift at zero cost. Add a container only when onboarding friction justifies it |

### 6.2 Mandatory vs optional

| Mandatory | Optional | Not recommended for this scope |
| --------- | -------- | ------------------------------ |
| Git | WSL2 (Windows) | Full VMs for daily dev |
| Node.js 24 LTS (or 22 ≥ 22.22.3) | Dev Containers | Kubernetes / Compose stacks |
| Bun (or one consistent fallback: pnpm) | Docker (deploy rehearsal) | Mixing package managers |
| A code editor | Cloud dev env | Running the repo from `/mnt/c` inside WSL2 |

---

## 7. New developer setup (minimal)

```bash
# 1. Clone the repository (layer 1)
git clone <repo-url> coba-cv && cd coba-cv
#    Windows + WSL2: run this inside the WSL shell, under ~/ (not /mnt/c)

# 2. Install the pinned Node version (layer 2)
fnm install && fnm use          # reads .nvmrc → 24   (or: nvm install && nvm use)
node -v                         # must be >= 24.15.0

# 3. Install Bun (once per machine), see https://bun.com/docs/installation
bun -v

# 4. Install dependencies and start
bun install
bun run dev                     # http://localhost:3000 (Windows native: http://127.0.0.1:3000)

# 5. Before pushing (AGENTS.md definition of done)
bunx nuxi typecheck
bun run generate                # static site in .output/public
```

---

## 8. Optional workflows (use only when justified)

### 8.1 WSL2 (Windows)

**Use when:** native Windows HMR or installs are slow, or you want parity with a
Linux VPS. **Skip when:** you have 8 GB of RAM and native works fine.

1. `wsl --install` (Ubuntu), then reboot.
2. Inside WSL: install fnm, Node 24, Bun, Git, and clone into `~/projects`.
3. Open with VS Code (`code .`) using the WSL extension.
4. Optionally cap memory with `.wslconfig` (see §5).

### 8.2 Dev Containers

**Use when:** a team, class, or workshop needs identical toolchains or
one-click onboarding (including Codespaces). **Cost:** requires a container
engine, which is a VM on Windows/macOS.

Not committed by default, to keep the baseline lightweight. A minimal
`.devcontainer/devcontainer.json` if you need one:

```jsonc
{
  "name": "coba-cv",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:24",
  "features": { "ghcr.io/devcontainers/features/bun:1": {} },
  "forwardPorts": [3000],
  "postCreateCommand": "bun install",
  // keep Linux-native binaries out of the host folder
  "mounts": ["source=coba-cv-node_modules,target=${containerWorkspaceFolder}/node_modules,type=volume"],
  "customizations": { "vscode": { "extensions": ["Vue.volar"] } }
}
```

> Check that the image tag and the Bun feature are still current at
> <https://containers.dev> before adopting.

### 8.3 Docker (deployment rehearsal only)

**Use when:** you deploy the **Node server** build to a VPS or container host
and want to test it locally. A static deploy doesn't need Docker at all.

```dockerfile
# Build with Bun, run on Node LTS (matches Nuxt engines)
FROM oven/bun:1 AS build
WORKDIR /app
COPY package.json bun.lock ./
RUN bun install --frozen-lockfile
COPY . .
RUN bun run build

FROM node:24-slim
WORKDIR /app
COPY --from=build /app/.output ./.output
ENV PORT=3000
EXPOSE 3000
CMD ["node", ".output/server/index.mjs"]
```

---

## 9. Deployment compatibility (summary)

| Target | Command | Output | Notes |
| ------ | ------- | ------ | ----- |
| **Static hosting** (PRD default; also any VPS with Nginx/Caddy) | `bun run generate` | `.output/public/` | No Node needed on the server. The best fit for a CV |
| **Linux VPS, Node server** | `bun run build` → `node .output/server/index.mjs` | `.output/` | Use Node 24 LTS on the server. Manage the process with systemd or PM2, behind Nginx/Caddy |
| **Node-capable PaaS** | `bun run build` | Nitro preset auto-detected | See the Nuxt deployment docs for provider presets |

---

## 10. Trade-offs and limitations

- **Bun as package manager, Node as runtime**: you get fast installs, but there
  are two tools to install. The fallback (pnpm) requires switching the lockfile
  for the whole fork.
- **WSL2**: better Linux parity and file watching than native Windows, but it
  reserves VM memory and makes `/mnt/c` access slow. Not worth it on 8 GB unless
  native is unusable.
- **Containers**: reproducibility costs startup time, disk, and (on
  macOS/Windows) a hidden VM. Watch out for native-binary mismatches with
  bind-mounted `node_modules`.
- **Hardware comfort ratings are qualitative.** They come from architectural
  factors (virtualization layers, filesystem boundaries, concurrent processes),
  not from benchmarks of this repo. Measure on your own machine if it matters.
- **Version facts age.** Recheck Node LTS status and Nuxt `engines` when
  upgrading Nuxt (`npm view nuxt engines`).

---

## 11. References

- Nuxt installation & prerequisites: <https://nuxt.com/docs/4.x/getting-started/installation>
- Nuxt deployment: <https://nuxt.com/docs/4.x/getting-started/deployment>
- Nuxt prerendering / `nuxi generate`: <https://nuxt.com/docs/4.x/getting-started/prerendering>
- Node.js release status: <https://nodejs.org/en/about/previous-releases>
- Bun installation & platform support: <https://bun.com/docs/installation>
- WSL file-system performance guidance: <https://learn.microsoft.com/windows/wsl/filesystems>
- WSL `.wslconfig` options and defaults: <https://learn.microsoft.com/windows/wsl/wsl-config>
- Dev Containers specification: <https://containers.dev>
- VS Code WSL / Dev Containers: <https://code.visualstudio.com/docs/remote/wsl>, <https://code.visualstudio.com/docs/devcontainers/containers>
- Installed package `engines`: `node_modules/nuxt/package.json` (`nuxt@4.6.1`)
