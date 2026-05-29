# Result: cloud-manager/polished-squirrel

## Outcome
SHIPPED

## What was delivered

1. **Infinite await_merge loop** — removed 30-minute timeout from `ops/ship.go`; `await_merge` now polls every 30s forever (with initial 30s sleep after PR open) until all PRs are merged, then transitions.

2. **open_browser flag** — added `OpenBrowser bool` to `ShipConfig` in `internal/config/config.go`; `ops/ship.go` opens each PR URL in the default browser (`open` on macOS, `xdg-open` on Linux) after `ship.Run` succeeds when `open_browser: true`.

3. **config.yaml updated** — `open_browser: true` added under `ship:` in `workflow-configuration/config.yaml` and synced to `~/.hv/config.yaml`.

4. **MCP hint updated** — `ship.ts` await_merge hint now says "hv_ship is polling — it will transition automatically once all PRs are merged."

## Tests
All pass: `go test ./...`, `go build ./...`, `npm run build`.
