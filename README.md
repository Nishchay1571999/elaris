# Elaris

Elaris is an AI assistant and software development platform. It is planned as three products that share one identity and, eventually, one Rust agent core:

| Product | What it is | Status |
| --- | --- | --- |
| **Elaris Web** | AI chat in the browser: free basic chat, paid browsing and advanced reasoning, persistent history | **UI milestone implemented** (`apps/web`) |
| **Elaris CLI** | Terminal-first coding agent in Rust, with Neovim, LSP and Tree-sitter integration | Planned |
| **Elaris Desktop** | Tauri v2 app combining chat and coding, with embedded Neovim and a Ghostty-based terminal | **First build implemented** (`apps/desktop`) |

## Repos

Elaris is split into independent git repos, so each can be public or private on its own:

| Repo | Cloned to | Contains |
| --- | --- | --- |
| `elaris` (this one) | `.` | Workspace: shared docs, brand assets, scripts that run across apps |
| `elaris-desktop` | `apps/desktop` | Desktop app, its Rust crates and the agent sidecar. Builds on its own |
| `elaris-web` | `apps/web` | Web app. Builds on its own |

This repo ignores `apps/*/`, so each app's history stays in its own repo. Nothing here is needed to build an app; it only ties them together.

## Getting started

```bash
git clone git@github.com:Nishchay1571999/elaris.git
cd elaris
pnpm bootstrap    # clones every app repo you can access into apps/, then installs each
```

Or clone just one app and pull in the rest later from inside it: every repo carries `scripts/elaris-repos.sh` (`pnpm repos …`):

| Command | What it does |
| --- | --- |
| `pnpm repos list` | Show every repo and whether it's cloned |
| `pnpm repos clone [desktop\|web]` | Clone missing repos (all by default). From a standalone app clone, first clones this workspace to `../elaris` |
| `pnpm repos pull` | `git pull --ff-only` in every clone |
| `pnpm repos status` | `git status` of every clone |

Repos you can't access are skipped, so outside contributors to an open-source app get just that app. Use `ELARIS_GIT_BASE` to clone from a fork or another host.

### Commands

These run from the workspace root and delegate to the app. Inside an app repo, run the same names there.

| Command | What it does |
| --- | --- |
| `pnpm dev` / `build` / `start` / `lint` / `typecheck` | Web app (`apps/web`) |
| `pnpm desktop:dev` / `desktop:app` / `desktop:build` / `desktop:install` / `desktop:typecheck` | Desktop app (`apps/desktop`) |

See each app's README for what it does and how it works.

## Open-sourcing an app

Each app is already its own repo with its own history, so open-sourcing one means:

1. Add a `LICENSE` to that repo and set `license` in its `package.json` (and `license` under `[workspace.package]` in `apps/desktop/Cargo.toml`, currently `UNLICENSED`).
2. Make that GitHub repo public. Leave the others private.

Nothing in an app repo refers to private code: it builds from its own files, and its `pnpm repos clone` simply skips repos the reader can't access.

## Layout

```text
elaris/
├── .elaris-workspace     # marks the workspace root for elaris-repos.sh
├── logo.png              # Official brand mark (source of truth)
├── docs/architecture.md  # how the products fit together, and the roadmap
├── scripts/elaris-repos.sh
└── apps/                 # each folder is its own repo (git-ignored here)
    ├── desktop/          # elaris-desktop
    └── web/              # elaris-web
```

`scripts/elaris-repos.sh` is copied into each app repo. When you change it, copy it to `apps/*/scripts/` too so every repo stays in step.

## Documentation

- [docs/architecture.md](docs/architecture.md): how the products fit together and the Rust-first roadmap
- `apps/web/docs/`: web architecture and the design system
- `apps/desktop/README.md`: how the desktop app and its agent work
