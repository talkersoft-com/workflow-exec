# Safe Install Config

## What this workflow does
`make install-config` currently does a blind `cp -R .hv/. ~/.hv/` that silently overwrites any local edits the user has made in `~/.hv/`. This workflow changes the behavior so the default is safe: only copy files that don't exist at the destination, warn about skipped conflicts, and require an explicit `FORCE=1` flag to overwrite.

## Read before starting
- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `../../common/toolkit/hive-deck.md` — hv MCP call patterns

## Constraints
- `make install` must still work end-to-end — it calls `install-config`
- `make install-config FORCE=1` must behave like the old `cp -R` (full overwrite)
- No external tools beyond standard shell (`cp`, `diff`, `printf`) — the Makefile must work on a stock macOS install
