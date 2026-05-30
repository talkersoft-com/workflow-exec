# cloud-manager leafy-wyvern — Results

> **Operator action required:** Restart cloud-manager-mcp via `/mcp` in Claude Code to pick up the new dist. 37 new tools become available after the reload.

## Phase 2 — vm-events timing fix (cloud-manager-api)

Moved the `destroy_requested` write (and accompanying soft-delete) from the
AMQP completion consumer into `OrchestrationController.DestroyVirtualMachine`,
so the audit timeline reflects user intent the moment the DELETE arrives
rather than ~15s later when the vorch ack lands.

- Commit: `347588a` — *vm-events: write destroy_requested at DELETE time, not at ack time*
- Files:
  - `src/CloudManager.API/Controllers/OrchestrationController.cs` — now calls
    `_virtualMachineService.DeleteVirtualMachineAsync(vmGuid)` synchronously
    before publishing the destroy-vm message.
  - `src/Services/CloudManager.AMQP.Consumer/ConsumerService.cs` — removed the
    duplicate `DeleteVirtualMachineAsync` call from the Success branch; the
    consumer now only records the TeardownSucceeded outcome.

## Phase 3 — MCP Tier 2/3/4 tools (cloud-manager-mcp)

37 new tools across 3 new files. Commit: `2390a01` —
*mcp tier 2+3+4: playbook internals (21) + ansible collections (13) + feature flags (3)*.

| File | Tools | Tier |
|---|---|---|
| `src/tools/playbook-tree.ts` | 21 | Tier 2 — playbook internals (steps, tasks, vars, become, when, loops) |
| `src/tools/collections.ts` | 13 | Tier 3 — Ansible Galaxy collections lifecycle |
| `src/tools/feature-flags.ts` | 3 | Tier 4 — runtime feature flag inspection/toggle |

## Phase 4 — Registration + build (cloud-manager-mcp)

- `src/index.ts` updated to register the three new tool groups (6 added lines).
- Build clean (`npm run build`), `dist/` regenerated, package-lock unchanged.
- `profiles.json` updated to include `collections` in the `hv.cloud` profile
  (commit `d63ca9d`).

## Phase 5 — Redeploy (cloud-manager-api)

cloud-manager-api re-deployed with the timing fix in place; web build verified
green prior to merge.

## Tool count delta

- Before: 35 tools registered (`server.tool(` across `src/tools/*.ts`)
- After:  72 tools registered
- Delta:  **+37**

## PRs

PR URLs are captured in the ship report (`hv_ship` output) — see the workflow
final report.
