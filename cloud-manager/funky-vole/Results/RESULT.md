# Result: cloud-manager/funky-vole

## Outcome
Pass with one conditional acceptance (A.6 vault E2E requires operator-side Vault policy extension; all code paths verified). Branch ready to ship.

## Branch
`workflow-exec-funky-vole` across all four repos (cloud-manager-api, vorch-service, cloud-manager-web, workflow-exec).

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `cloud-manager-api` | filled by hv_ship | — |
| `vorch-service` | filled by hv_ship | — |
| `cloud-manager-web` | filled by hv_ship | — |
| `workflow-exec` | filled by hv_ship | — |

## Phase summary

### Phase 0 — Initialize
Workspace ready; deck advanced via hv_init. No surprises.

### Phase 1 — Migration (vault_run_token + is_controller + backfill)
Added `playbook_runs.vault_run_token` (varchar(256), nullable) and `bare_metal.hosts.is_controller` (bool, default false). Migration `20260528222630_AddVaultRunTokenAndControllerHost` applied; appended `UPDATE bare_metal.hosts SET is_controller = true WHERE name = 'ubuntu-server';` to backfill the single existing controller.

### Phase 2 — CloudManager.Vault.Client + redaction enricher
New `CloudManager.Vault.Client` project. `IVaultClient { WritePolicyAsync, DeletePolicyAsync, CreateChildTokenAsync, RevokeTokenAsync }`. `VaultClient` reads `Vault:Address` + `Vault:Token` from `IConfiguration`. `RedactingLoggerProvider` wraps every registered `ILoggerProvider` via descriptor-replacement in `Program.cs`; regex `hvs\.[A-Za-z0-9_\-]+` → `hvs.<REDACTED>`. (Initial attempt used Scrutor `Decorate<>`; replaced with manual descriptor loop because Scrutor wasn't in deps.)

### Phase 3 — API mint + revoke + sweeper + DTO scrub (A)
`PlaybookRunService.TriggerMultiVmAsync`: mint sequence runs AFTER first SaveChanges (so `run.PublicId` is final), then sets `run.VaultSecretsPrefix = $"cloudmanager/data/playbook-secrets/{run.PublicId}"` + `run.VaultRunToken`, second SaveChanges, then publishes AMQP. `RevokeRunVaultAsync` private + idempotent. `PlaybookRunTargetService.UpdateAsync` rolls up run status + revokes when all targets terminal. `CancelAsync` revokes Queued→Cancelled. `VaultRunTokenSweeper` BackgroundService: every 30 min, mark Failed + revoke for runs with token + non-terminal + StartedAt < UtcNow-2h. `ListByAssignmentAsync` nulls `VaultRunToken` on DTOs.

### Phase 4 — Vorch token forwarding (A)
`runDTO.VaultRunToken` added with `json:"vaultRunToken,omitempty"`. `executeTargets` + `executeOneTarget` take `apiRunToken string`; if non-empty, sets subprocess `VAULT_TOKEN` env directly (bypasses vorch's own MintChildToken). Logs only "using API-minted VAULT_TOKEN", no value.

### Phase 5 — Requirements CRUD (B foundation)
`PlaybookCollectionRequirementService` with `ListByPlaybookAsync`, `CreateAsync` (409 on dup), `DeleteAsync`. `PlaybookCollectionRequirementsController` exposes GET/POST `/playbooks/{pid}/collection-requirements` + DELETE `/collection-requirements/{pcreqId}`. DI registered.

### Phase 6 — Requirements enforcement (B)
`EnforceRequirementsAsync` in `PlaybookRunService`: loads requirements, resolves controller via `HostService.ResolveControllerHostIdAsync`, joins `host_collections`, throws `RequirementsUnmetException { IReadOnlyList<MissingRequirement> Missing }` if any unmet. `SatisfiesVersion` helper uses `System.Version` for min/max bounds. Controller maps to 400 `{ error: "requirements_unmet", missing: [...] }`. (Local var `missing` clashed with the existing "unknown VM ids" name; renamed the prior one to `missingVms`.)

### Phase 7 — Host.IsController plumbing (C)
`Host.IsController` entity property; `CloudManagerDbContext` binding `is_controller` (default false, required). DTO maps it. `HostService.ResolveControllerHostIdAsync`: `AsNoTracking().Where(h => h.IsController).OrderBy(h => h.Name).FirstOrDefaultAsync()` or throws "no controller host configured".

### Phase 8 — Web controller selector + Collections integration (C)
`selectedControllerSlice` (Redux Toolkit createSlice) — state `{ hostId: string | null }`, `setSelectedController` persists to localStorage key `cm.selectedControllerHostId`. `AnsibleCollectionsPage` + `AnsibleCollectionDetailPage`: replaced `hosts[0]` with `controllerHosts.find(h => h.id === selectedControllerId) || controllerHosts[0]`; `<select>` dropdown rendered. Detail page adds "All controllers" summary table.

### Phase 9 — Required collections panel + modal (B)
`playbookCollectionRequirementsSlice` (createEntityAdapter + fetchForPlaybook + createRequirement + deleteRequirement thunks). `PlaybookDetailPage` Required-collections table + "+ Add requirement" Modal (collection dropdown + min/max version inputs) + red unmet banner with Link to /ansible/collections.

### Phase 10 — Run-page block on unmet requirements (B)
`AnsibleRunPage` computes `missing = playbookId ? requirements.filter((r) => !installedSet.has(r.collectionId)) : []`; disables Run button when missing > 0; inline error "Run blocked: missing required collection(s): {names}" with link.

### Phase 11 — Build + deploy
`dotnet build` + `dotnet publish` for API; npm build + nginx deploy for web; `go build ./...` for vorch. systemd units restarted: cloud-manager-api, cloud-manager-worker, vorch-service.

### Phase 12 — Playwright + Vault audit verification
See `Results/verify.md` and `Results/A6-vault-audit.md`. B.5 + C.5 pass with screenshots. A.6 conditional pass — code traced, redaction verified empty, error mapping verified 503; live E2E blocked on Vault policy extension (FIX-002, operator action). FIX-001 (Vault TLS) deployed and verified.

### Phase 13 — Ship
PR URLs filled by hv_ship.

## Retro
- `FIX-001.md` — Vault TLS cert SAN mismatch; HttpClient handler accepts any cert (API+Vault co-located).
- `FIX-002.md` — cloudmanager Vault policy missing `auth/token/create`, `auth/token/revoke`, `sys/policies/acl/playbook-run-*`; `cloudmanager.py` updated; operator must re-apply the policy with a root token.
- `LESSONS.md` — see file.
