# Elaris

Elaris is an AI assistant and software development platform. There are two apps today:

| App | What it is | Repo |
| --- | --- | --- |
| **Elaris Desktop** | A macOS app for chatting with an AI coding agent (Claude, via the Claude Agent SDK) that works in a folder on your computer. It also has terminals, git, GitHub pull requests and issues, saved chats you can reuse as context, and control of your desktop, Android emulators and iOS simulators | [`elaris-desktop`](https://github.com/Nishchay1571999/elaris-desktop) → `apps/desktop` |
| **Elaris Web** | A marketing home page and a browser chat UI. The chat streams replies from a local mock rather than a model, and keeps your history in the browser | `elaris-web` → `apps/web` |

This repo is the **workspace**. It holds the brand assets and the scripts that tie the app repos together. Each app is its own git repo with its own history, and builds without this one.

## Elaris Desktop

A Tauri v2 app: a React UI, Rust for everything native, and a Node sidecar running the Claude Agent SDK. The [desktop README](https://github.com/Nishchay1571999/elaris-desktop#readme) describes every feature in detail.

- **Agent chat**
  - **Models:** Claude Opus 5.5, Sonnet 5.5 or Haiku 5.5, or an offline mock agent.
  - **Modes:** Agent, Ask (no file changes) and Plan (propose a plan for approval).
  - **Permissions:** Ask before edits, Accept edits, Auto, or Bypass. Approvals can be given once, or always for the folder.
  - **During a reply:** the agent can ask you multiple-choice questions, start subagents (each opens in its own tab), and you can switch model, mode and permissions.
  - **Tool calls** stream into the chat; expand one to see its diff, command output or file contents.
- **Composer**
  - Send images by picking, pasting or dragging them in.
  - Stash prompts to use later.
  - Queue messages while a reply runs.
  - Attach saved chats with `#ctx-…`, and saved PRs and issues with `%pr-…` or `%issue-…`.
- **Saved contexts.** Save any chat (every prompt, reply, tool call and terminal output) and attach it to a later chat. Large contexts are read page by page or searched by the agent, through SQLite full-text search.
- **Terminals.** Your shell runs in tabs beside the chat. The agent runs commands in tabs you can watch and type into.
- **Git.** See changes and diffs against HEAD, commit, commit and push, switch and check out branches.
- **GitHub, through the `gh` CLI**
  - Connect, with a guided install and device-flow sign-in.
  - Pull request and issue lists across all your repos, with filters.
  - Full PR pages: conversation, review threads, files with a viewed state, checks and commits.
  - Merge, review, comment, check out, and create pull requests.
- **Desktop control** (macOS). The agent takes screenshots and clicks, types, scrolls and opens apps.
- **Emulators and simulators.** The agent drives Android emulators (`adb`) and iOS simulators (`simctl`, AXe) with low-token tools:
  - screens read as numbered lists of elements;
  - batched actions;
  - filtered app logs;
  - app files, plus SQL on the app's databases.
- **Code highlighting.** highlight.js first, then language-server semantic tokens. The servers are found through your Neovim config, and Mason can install missing ones.
- **Customizations**
  - MCP servers: a catalog of 14, saved in `~/.claude.json`.
  - Skills from skills.sh.
  - An editor for the `CLAUDE.md` files, with token costs.
  - Allow, Ask or Off for each tool, saved in `~/.claude/settings.json` so Claude Code follows them too.
- **App**
  - A command palette (⌘K).
  - Rebindable shortcuts.
  - Themes, accent colours, text size and density.
  - Project logos.
  - A sidebar of chats from every project, with live status, snooze, close, archive and a deleted shelf.

## Elaris Web

A Next.js 16 app (React 19, Tailwind CSS 4, shadcn/ui, Zustand, Zod).

- **`/`: home page.** Describes Elaris. It has a "Try it" box that opens the chat with your prompt ready to send, and an interactive illustration of the apps.
- **`/chat`: chat.** On a `chat.` subdomain it is also served at `/`.
  - **Conversations:** new chat, search, grouping by date (Today, Yesterday, Previous 7 days, Older), rename and delete.
  - **Replies** stream in and can be stopped. A reply keeps streaming while you switch to another conversation.
  - **Messages:** edit and resend, regenerate, and copy. Replies render as Markdown with highlighted, copyable code blocks.
  - **Settings:** light, dark or system theme, a response-animation switch, clearing history, and a list of keyboard shortcuts (⌘K search, ⇧⌘O new chat, ⌘B sidebar).
  - **Storage:** conversations, drafts and settings are kept in the browser's `localStorage`, versioned and checked with Zod.
- **Demo mode.** Replies come from a deterministic local mock behind a `ChatTransport` interface. The UI labels this clearly. There is no backend, no accounts and no model provider. The capability menu shows *Elaris Standard*; *Web browsing* and *Advanced reasoning* appear disabled.

## Getting started

```bash
git clone git@github.com:Nishchay1571999/elaris.git
cd elaris
pnpm bootstrap    # clones every app repo you can access into apps/, then installs each
```

Or clone just one app, and pull in the rest later from inside it. Every repo carries `scripts/elaris-repos.sh` (run it with `pnpm repos …`):

| Command | What it does |
| --- | --- |
| `pnpm repos list` | Shows every repo and whether it's cloned |
| `pnpm repos clone [desktop\|web]` | Clones missing repos (all of them by default). From a standalone app clone, first clones this workspace to `../elaris` |
| `pnpm repos pull` | Runs `git pull --ff-only` in every clone |
| `pnpm repos status` | Shows `git status` for every clone |

Repos you can't access are skipped. Set `ELARIS_GIT_BASE` to clone from a fork or another host.

### Commands

These run from the workspace root and pass through to the app. Inside an app repo, run the same names there.

| Command | What it does |
| --- | --- |
| `pnpm dev` / `build` / `start` / `lint` / `typecheck` | Web app (`apps/web`) |
| `pnpm desktop:dev` / `desktop:app` / `desktop:build` / `desktop:install` / `desktop:typecheck` | Desktop app (`apps/desktop`) |

The desktop app needs Node.js 20.9 or newer, pnpm 10, Rust and the [Tauri v2 prerequisites](https://v2.tauri.app/start/prerequisites/). The web app needs Node.js 20.9 or newer and pnpm 10.

## Layout

```text
elaris/
├── .elaris-workspace     # marks the workspace root for elaris-repos.sh
├── logo.png              # the official brand mark (source of truth)
├── docs/
├── scripts/elaris-repos.sh
└── apps/                 # each folder is its own repo (git-ignored here)
    ├── desktop/          # elaris-desktop
    └── web/              # elaris-web
```

This repo ignores `apps/*/`, so each app's history stays in its own repo. `scripts/elaris-repos.sh` is copied into each app repo; when you change it, copy it to `apps/*/scripts/` too so every repo stays in step.
