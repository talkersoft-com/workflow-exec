# Result: cloud-manager/neon-marmot

## Outcome
SHIPPED

## Branch
`neon-marmot`

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `execution-workflow` | TBD — filled from hv_ship output | — |

## Phase summary

### Phase 0 — Initialize
All 14 repos already on `neon-marmot`, all clean.

### Phase 1 — Implement force_overwrite
- Added `ForceOverwrite *bool` to `ClaudeSettings` in `internal/config/config.go`
- Added `ShouldOverwrite()` helper (nil or true → overwrite; false → additive merge)
- Updated `writeSettingsFile` in `internal/claude/claude.go` to skip reading existing file when `ShouldOverwrite()` is true
- Added `force_overwrite: true` to `.hv/config.yaml.example` with documentation
- Followed `*bool` pattern already established by `DeckConfig.EnableRootMCP`

### Phase 2 — Build
`CGO_ENABLED=0 go build ./...` — clean, no errors.

### Manual steps
hive-deck-pro pushed to `warm-leviathan`: commit `e45355a`
