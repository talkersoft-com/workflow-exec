# Result: hive-deck-pro/drifting-toadstool

Shared config layer + canonical path tokens (R7.3–R7.6). Executed on branch
`fragrant-hideaway`, 2026-06-11. All five tasks completed; every TEST file passed.

## What shipped

### R7.5/R7.6 — token engine (Task 0001)
- `internal/config/paths.go`: `ExpandPath(declaringFile, value, ws)` — `{{WorkspaceRoot}}`
  (canonical), `{{DecksRoot}}` (legacy alias kept indefinitely), `~`/`~/x`, relative paths
  joined to the directory of the declaring file, absolute unchanged; unresolved deck-scoped
  tokens (`{{DeckRoot}}`) pass through for later resolution. `ExpandTilde` extracted;
  `ExpandRoot` converged onto it.
- Wired into `plan_folder`/`exec_folder` (ops/workflow.go `resolveWorkflowPath`, ops/ship.go)
  and `deck_config`. `Setup.SourcePath` records the declaring config.yaml.
- `internal/mcp/mcp.go`: `resolveArg`/`resolveEnvVal` accept `{{WorkspaceRoot}}` and `~`;
  npm/flag passthrough untouched.

### R7.3/R7.4 — shared layer (Task 0002)
- `Setup.DeckConfig` (`deck_config:` key) expanded via the token engine relative to config.yaml.
- `internal/config/layer.go`: `layerPaths`, `DeckConfigRoot`, `ConfigRoots`,
  `findConfigFileLayered`, `findInConfigSubdirLayered`, `FindDecksDirs`.
- Per-FILE fallback (first hit wins wholesale, no key merging) in: deck yamls, workflow
  extensions, modules.yaml, claude-profiles.yaml, gitignore-rulesets.yaml, mcps.yaml
  (mcps.yaml kept machine-first WITH layer fallback for consistency — documented in README).
- `config.yaml` never falls back — machine identity only.
- Union scans, machine wins on filename: `hv list decks`, workflow registry
  (`LoadRegistry` over all roots with shadowing dedupe), `listWorkflowNames`, and the
  fragment chain (machine deck-fragments → layer deck-fragments → builtin).

### Machine identity via HV_PROFILE (Task 0003)
- `$HV_PROFILE` set AND `$HV_HOME/profiles/$HV_PROFILE/config.yaml` exists → that file is the
  machine config and its directory the machine root; else root config.yaml exactly as today.
- README: "Machine truth vs shared truth" section — model, resolution order, token table,
  HV_PROFILE selection, override-by-copying recipe.

## Verification evidence

### Golden back-compat (TC-001, single-layer, real workflow-configuration)
Captured with a pre-change binary (HEAD 4734667) and re-run post-change — all byte-identical:

| Capture | Result |
|---------|--------|
| `hv list decks` | IDENTICAL (after FIX-001) |
| `hv plan --list hive-deck-pro` | IDENTICAL |
| `hv promote --list hive-deck-pro` | IDENTICAL |
| `hv plan plan@hive-deck-pro` (assembled plan, 224 lines) | IDENTICAL |
| `hv promote feature@hive-deck-pro` (assembled workflow, 322 lines) | IDENTICAL |
| `hv mcp hive-deck-pro` + resulting root `.mcp.json` | IDENTICAL |

### Two-layer demo (TC-002, temp HV_HOME copy of the real config)
- Shared truth moved to `base/`, thin machine config with `deck_config: ./base`:
  decks list, registry, and plan assembly all resolve from the layer.
- Machine-side `decks/hive-deck-pro.yaml` override shadows the layer copy
  (`hv list repos` shows the override's single repo).
- Real workflow-configuration: zero diffs (git status clean throughout).

### Unit tests
`go test ./...` green. New coverage: expandPath form matrix, mcps token/alias equivalence,
layer shadow matrix (deck/modules/profiles/rulesets/mcps/fragments), registry layered dedupe,
config.yaml exemption, single-layer short-circuit, HV_PROFILE selection + fallbacks.

### Fixes
- `Retro/FIX-001.md` — decks listing sort regression (stem sort vs filename sort) caught by
  the golden diff; fixed by sorting filenames with suffix as before.

## Artifacts rebuilt and installed
- Go binary: `make install` → `~/go/bin/hv` (installed only after all tests + goldens passed).
- MCP dist: `npm run build` in `hive-deck-pro/mcp/` → `dist/` regenerated.
- **Operator note: restart the hive-deck MCP (`/mcp`) to pick up the new binary/dist.**

## PRs
Shipped via `hv_ship stage:"exec"` from branch `fragrant-hideaway`
(commit "feat: shared config layer + canonical path tokens (R7.3-R7.6)").
Expected PRs: `hive-deck-pro` (code) and `workflow-exec` (this orchestration folder).
PR links are reported in the ship output and the operator chat report.
