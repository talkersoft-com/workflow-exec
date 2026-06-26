# RESULT — cloud-manager/nostalgic-jetplane (Secret Bindings #3)

> **Operator action required:** Restart cloud-manager-mcp via `/mcp` in Claude Code to pick up the
> two new tools (`cloud_vm_secret_binding_list`, `cloud_blueprint_secret_binding_update`). The deploy
> script rebuilt the MCP dist but does **not** restart the running server, so until you `/mcp` reload
> these tools are not yet callable from Claude.

## Outcome
**SUCCESS.** All five phases implemented, every Test file passed, api + mcp + web build clean, api
deployed and healthy, mcp dist rebuilt. The full attach → update → provision → list-usage round-trip
was verified against the **live API**.

## Branch
`coppery-canteloupe` — execution branch minted by `hv_next` in the warmup (from origin/hive across
all 16 repos). NOTE: `deck.md` originally pre-recorded `rakish-coldfront`; the warmup actually minted
`coppery-canteloupe`, which Task 0000 recorded as the real execution branch (no second branch created).

## Pull Requests
`hv_integrate` opens feature → hive PRs for the two repos with code changes (the planning repos carry
the orchestration record). Links:

| Repo | PR |
|------|----|
| cloud-manager-api | _see hv_integrate output / reported in chat_ |
| cloud-manager-mcp | _see hv_integrate output / reported in chat_ |

## Code changes
### cloud-manager-api (Phase 1)
- `VmSecretBindingUsage` DTO (metadata + resolved Vault **path**, never a value).
- `IVmSecretBindingService.ListUsageForVmAsync(string vmPublicId)` + impl in `VmSecretBindingService`
  — joins `vm_secret_bindings` → `secret_bindings` (inner join through the soft-delete query filter),
  resolves the path (static → `Secret.vault_path`; templated → `path_template` with `{vm_public_id}`
  substituted; **never reads Vault**), public ids only, `KeyNotFoundException` → 404.
- `GET /api/v1/vm/{publicId}/secret-binding` on `VirtualMachineController` (absolute route mirroring
  the blueprint↔binding endpoints), `IVmSecretBindingService` injected.

### cloud-manager-mcp (Phases 2 & 3)
- `cloud_vm_secret_binding_list` (in `vms.ts`) — wraps `GET /api/v1/vm/{id}/secret-binding`; input
  `id` (vm_…); returns usage rows; no value.
- `cloud_blueprint_secret_binding_update` (in `secret-bindings.ts`) — wraps the **existing**
  `PATCH /api/v1/blueprint/{id}/secret-binding/{bsbId}`; inputs `id`, `bsbId`, optional `credName`,
  optional `position`; **send-only-provided** body.

No DB / vorch / web code changes. **No EF migration added** — `dotnet ef database update` was NOT run.

## Phase / Test summary
| Phase | What | Test result |
|-------|------|-------------|
| 0000 | Setup: branch recorded; deps confirmed (`vm.vm_secret_bindings` table + PATCH endpoint) | PASS (TC-001…004) |
| 0001 | API read endpoint | PASS — TC-001 full row, TC-002 `[]`, TC-003 404, TC-004 no value/no Guid, TC-005 soft-delete excluded, TC-006 build |
| 0002 | MCP `cloud_vm_secret_binding_list` | PASS — wrapped endpoint returns rows verbatim (no value); single-`id` input; build clean |
| 0003 | MCP `cloud_blueprint_secret_binding_update` | PASS — TC-001 credName, TC-002 position reorder, TC-003 send-only-provided, TC-004 no API change, TC-005 build |
| 0004 | Round-trip + build + deploy | PASS — attach→update→provision→list-usage green; build all clean; api healthy; mcp dist rebuilt |

### Live round-trip evidence (Phase 4)
- attach `sb_W6HN0WCP05` (static, → `cloudmanager/data/manual/njp-e2e`) to `bp_29ZRV0YERX`, credName `DBPASSWD`.
- PATCH credName/position verified send-only-provided (on `bp_VEEC7P60WX`, two bindings, reorder confirmed).
- provision `bp_29ZRV0YERX` → `vm_89PF4XWDCZ` (orchestrationStatus → Completed).
- `GET /api/v1/vm/vm_89PF4XWDCZ/secret-binding` →
  ```json
  [{"publicId":"vsb_RSN2EMMPM6","credName":"DBPASSWD","secretBindingId":"sb_W6HN0WCP05",
    "secretBindingName":"njp-e2e-static","resolvedVaultPath":"cloudmanager/data/manual/njp-e2e",
    "createdAt":"2026-06-26T21:27:46.413237Z"}]
  ```
  Metadata + **path** only — no secret value; ids are `vsb_…`/`sb_…`, no Guid.
- edge cases: unknown VM → 404; VM with no bindings → `[]`; soft-deleted binding excluded then restored.

## MCP transport note
The two **new** MCP tools cannot be invoked over MCP until the operator `/mcp` reload (above). Their
behavior was verified against the live API through the exact endpoints/bodies they wrap (direct HTTP),
plus a clean `--target mcp` TypeScript build and the existing tool-registration pattern.

## Deploy
| Target | Action | Smoke |
|--------|--------|-------|
| api | `dotnet publish` → swap `/opt/cloud-manager-api` → restart service | `GET http://localhost:5250/api/v1/Host/list` → **HTTP 200**; `cloud_health_check` → **Healthy** |
| mcp | dist rebuilt (`tsc`) — running server **not** restarted | operator `/mcp` reload required |

## Test artifacts left in the live system (not shipped by git)
- VMs: `vm_89PF4XWDCZ` (e2e success, has the usage row), `vm_PPR04V26XF` (failed templated-binding
  attempt — see Retro/FIX-001).
- Blueprints: `bp_29ZRV0YERX` (static, provisioned), `bp_VEEC7P60WX` (templated, PATCH tests).
- Bindings: `sb_W6HN0WCP05` (static), `sb_P2BC7Y4JJW`, `sb_62PAR086VS` (templated).
- Secret: `sec_PS6HKC219V` (`cloudmanager/data/manual/njp-e2e`).
These can be torn down at the operator's discretion; they are fixtures, not part of the code change.
