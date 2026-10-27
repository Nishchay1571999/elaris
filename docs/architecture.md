# Elaris architecture

How the Elaris products fit together and where they are going. Anything under **Planned** is direction only. None of it has been built.

## Current state

Each product is its own repo, cloned into `apps/` (see the README):

- **Elaris Web** (`apps/web`): Next.js chat UI with a mock transport. See `apps/web/docs/architecture.md`.
- **Elaris Desktop** (`apps/desktop`): Tauri v2 app; its Rust crates live in `apps/desktop/crates`. See `apps/desktop/README.md`.

## Planned (not implemented)

### Rust-first platform

Native and backend systems will be written in Rust, and no Go, Python or Node backend services are planned. Node.js is used for front-end tooling, Next.js, and one exception: Elaris Desktop runs the Claude Agent SDK (TypeScript/Python only) in a local Node sidecar that Rust supervises over stdio (`apps/desktop/crates/agent-core/src/claude.rs`, `apps/desktop/sidecar`).

```text
apps/
├── web/               # Next.js / React (exists)
├── desktop/           # Tauri v2 + React UI + Rust commands
└── cli/               # Rust CLI binary

crates/                # in apps/desktop today; moves to a shared repo once the CLI needs it
├── agent-core/        # Agent loop, context management, orchestration
├── agent-protocol/    # Shared request/stream/event types (serde), mirrored to TS
├── agent-tools/       # Shell, file, Git, diagnostics tools; sandbox-aware
└── code-intelligence/ # LSP client + Tree-sitter parsing

services/
├── api/               # Rust (Axum): accounts, conversations, entitlements
└── model-gateway/     # Rust: provider routing, streaming, quotas
```

How these pieces relate:

- **One protocol.** `agent-protocol` defines the event stream. The web `ChatStreamEvent` union is intended to become a TypeScript mirror of it, so web, desktop and CLI speak the same events.
- **Web → services.** When `services/api` and `services/model-gateway` exist, the web app gets an HTTP/SSE `ChatTransport` adapter and a server-backed `Store`. Entitlements come from the API instead of `DEMO_ENTITLEMENTS`.
- **CLI and desktop share the agent.** Both link `agent-core`, `agent-tools` and `code-intelligence` directly. The desktop app exposes them through Tauri commands.
- **Desktop renders locally.** It reuses React components and domain types from `apps/web` once they are extracted into a shared package (only when the second consumer exists). It must not depend on the hosted web app to render its UI.
- **Neovim is an editor integration.** The agent drives Neovim over its RPC API, and agent business logic does not live in Neovim. The CLI must work in any terminal without Neovim.
- **Ghostty is a terminal renderer choice for the desktop app**, with a simpler renderer as a fallback. The Rust agent core does not depend on it.
- **Hosted code execution** will require isolated sandbox infrastructure before `agent-tools` can run anything on behalf of web users.

### Suggested order

1. Extract `agent-protocol` types and generate the TS mirror. Replace the hand-written `ChatStreamEvent`.
2. Build `services/model-gateway` with one provider, plus an SSE `ChatTransport` adapter in web.
3. Add accounts, server-side conversations and entitlements in `services/api`.
4. Build the CLI on `agent-core` and `agent-tools`, then the desktop app.
