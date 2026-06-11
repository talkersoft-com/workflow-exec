# Marketplace backend — Blueprints, Marketplace API, run sequencer, MCP tools (lucky-engineblock)

## What this workflow does
Implements plan 1 of the marketplace series: the Blueprint data model (Blueprint +
BlueprintPlaybook ordered join + BlueprintEvent audit log + VM.BlueprintId provenance), the
Blueprint/Marketplace API surface (CRUD, Draft→Published→Archived lifecycle, ordered playbook
management, marketplace listing, instantiate), the BlueprintRunSequencer (auto-runs the playbook
chain in blueprint order after provisioning completes), MCP marketplace tools, and seeded `jammy`
(image-only) + `postgres-jammy` blueprints. The web UI is plan 2 (`copper-kiosk`) — no web changes
here.

## Read before starting
- `deck.md` — scope, branch, pre-written hv MCP calls
- `Orchestrate/ORCH.md` — task list and execution instructions
- `../../common/toolkit/hive-deck.md` — hv MCP call patterns
- `../../../workflow-plans/cloud-manager/lucky-engineblock/PLAN.md` — entity tables, endpoint
  shapes, sequencer design, open-question defaults (defaults are DECIDED — do not relitigate)

## Constraints
- **No vorch-lib / vorch-service / porch changes.** The sequencer hooks the API-side status
  writeback consumers only. If that proves impossible, STOP and surface — do not touch vorch.
- **No cloud-manager-web changes** — UI is plan 2.
- Existing direct VM playbook attach/apply must keep working byte-identically (Phase 0005
  regression-checks it).
- Conventions are non-negotiable: camelCase wire JSON, public IDs only on the wire (`bp`, `bpp`,
  `bpev` prefixes registered in EntityPrefixRegistry), soft-delete + filtered unique indexes
  (VirtualMachine pattern), append-only BlueprintEvent (VmEvent pattern, no update/delete),
  AutoMapper/service/controller stack copied from the Playbook exemplar,
  `[RequireFeatureFlag("marketplace")]` on every new endpoint.
- EF migration must be reversible; apply to the live DB BEFORE deploying the API.
- Build after every phase (`python3 .cicd/build-cloud-manager.py --target api|mcp`); deploy before
  the final ship per the deploy fragment.
