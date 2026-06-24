# Agent Arcade Rebuild — execution workflow (coarse-corgi)

## What this workflow does
Rebuilds the deprecated `agent-arcade-studio` Electron app into the empty `agent-arcade`
repo with re-architected internals while keeping UI, assets, menus, the keyboard contract,
and observable behavior **1:1**. The Arcade control surface becomes vanilla JS + XState +
mitt; Studio becomes a React SPA (Zustand). Go binaries, the Electron main/preload/launcher,
the `~/.hv/agent-arcade.yaml` schema, and the npm release pipeline carry over unchanged (the
pipeline gains one esbuild JS-bundling stage; no DMG). End state: a published
`@talkersoft-com/agent-arcade` that behaves identically to the old app but fixes draft-loss
on navigation, state desync, and fire-and-assume operations.

## Read before starting
- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Execution/Exec.md` — full task list and execution instructions
- `../../workflow-plans/agent-arcade/coarse-corgi/PLAN.md` — the approved plan (source of truth)
- Reference implementation (read-only): `/Users/talker/workspace/agent-arcade/agent-arcade-studio`

## Constraints (hard rules — do not violate)
- **Paradigm boundary:** the whole system is vanilla JS EXCEPT Studio (React). NO React/JSX/
  React-state-tooling anywhere in the Arcade. XState + mitt are the only added Arcade libs.
- **1:1 parity:** UI, assets, menus, and the keyboard contract (`⌘⌥A ⌘D ⌘F ⌘W ⌘←/→ ^C Esc`)
  must match the reference. Reuse `arcade/renderer/index.html` + CSS **verbatim**.
- **Do not change** the Go binaries' transport contracts (NDJSON, `wezterm cli`), the Electron
  main/preload/launcher behavior (reuse as-is), or the `~/.hv/agent-arcade.yaml` schema.
- **Pipeline:** preserve the release mechanism; the ONLY sanctioned change is adding an esbuild
  `build:js` stage. **No DMG / electron-builder.** Keep the node-pty `spawn-helper` postinstall.
- **Package:** `@talkersoft-com/agent-arcade`, bin `agent-arcade`.
