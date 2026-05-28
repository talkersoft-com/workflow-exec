# hive-deck JSON CLI Refactor — Operations Contract

## What this workflow does

Extracts all hive-deck operation logic into an `ops/` Go library package. The cobra CLI keeps calling ops directly via Go imports (no change for human users). The `ops/` package also exposes a JSON dispatch entry point via stdin so the MCP can drive any operation with a JSON payload. The MCP runner is updated to use JSON payloads instead of arg arrays. An operations contract (CONTRACT.md + schema.json) is committed documenting the JSON interface. Functionally 1:1 with the current version.

## Read before starting

- `deck.md` — repos in scope, branch, hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `/Users/talker/workspace/cloud-manager/planning/workflow-plans/cloud-manager/meteoric-bison/PLAN.md` — full design with exact code shapes

## Constraints

- All existing `hv <subcommand>` commands must produce identical output after refactor
- `go test ./...` must pass at every phase before moving to the next
- `make install` must continue to work
- CONTRACT.md must NOT be gitignored — it is a committed project artifact
- No new features — structural refactor only
