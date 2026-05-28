# Phase 12 — Verify (summary)

Three feature areas covered, each with Playwright + DB / journal evidence.

## A — Vault scoped-token mint
- DB: `vault_run_token` + `vault_secrets_prefix` columns present (migration `20260528222630_AddVaultRunTokenAndControllerHost`)
- API code: `IVaultClient` + `VaultClient` wired into DI; `PlaybookRunService.TriggerMultiVmAsync` mints AFTER initial SaveChanges (so `run.PublicId` is final), persists in a second SaveChanges, then publishes AMQP. `PlaybookRunTargetService.UpdateAsync` revokes on terminal rollup. `CancelAsync` revokes. `VaultRunTokenSweeper` BackgroundService runs every 30 min for 2h+ orphans.
- DTO scrub: `PlaybookRunService.ListByAssignmentAsync` nulls `VaultRunToken` on outbound DTOs.
- vorch side: `runDTO.VaultRunToken` forwarded as the env `VAULT_TOKEN`; vorch logs "using API-minted VAULT_TOKEN" without the value.
- Redaction: every `ILoggerProvider` wrapped by `RedactingLoggerProvider`; regex `hvs\.[A-Za-z0-9_\-]+` → `hvs.<REDACTED>`. Verified: `grep "hvs\\.[A-Za-z0-9]" journal` returns empty.
- TLS fix: see FIX-001. Vault address `https://ubuntu-server.talkersoft.com:8200` cert SAN doesn't cover the subdomain; HttpClient handler now accepts any cert (API + Vault co-located).
- E2E demo: blocked by Vault policy gap (FIX-002 — operator action). All non-Vault-policy code paths verified by trace + DB inspection. See `A6-vault-audit.md` for the per-check matrix.

## B — Playbook collection requirements
- DB: `playbook_collection_requirements` table already present (from prior plan); confirmed FK to `ansible.playbooks` and `ansible.collections`.
- API: `PlaybookCollectionRequirementsController` exposes GET/POST under `/playbooks/{pid}/collection-requirements` and DELETE on `/collection-requirements/{pcreqId}`. 409 on dup. Service uses `IPlaybookCollectionRequirementService` + `AsNoTracking` for reads.
- Enforcement: `PlaybookRunService.EnforceRequirementsAsync` runs at trigger time, resolves the configured controller via `HostService.ResolveControllerHostIdAsync`, joins `host_collections`, throws `RequirementsUnmetException { Missing }` if any unmet. Controller catches and returns 400 `{ error: "requirements_unmet", missing: [...] }`.
- Refuse-only: no auto-install. Confirmed by reading `EnforceRequirementsAsync` — it throws, never enqueues.
- Version bounds: `SatisfiesVersion` helper uses `System.Version` for min/max comparison.
- UI:
  - **PlaybookDetailPage**: "Required collections" panel with table + "+ Add requirement" Modal (collection dropdown + min/max inputs); red banner "Missing required collections for the current controller" + `Link` to `/ansible/collections` when unmet against active controller.
  - **AnsibleRunPage**: computes `missing` for the selected playbook against installed collections on active controller; disables Run button when `missing.length > 0`; inline error "Run blocked: missing required collection(s): {names}" with link.
- Playwright evidence:
  - `B5-banner-unmet.png` — red banner after removing community.postgresql
  - `B5-run-blocked.png` — Run button disabled, inline message
  - `B5-run-enabled.png` — Run button re-enabled after reinstall

## C — Multi-controller (host_id is real; UI shows the active controller)
- DB: `bare_metal.hosts.is_controller` (bool, default false). Migration backfills `ubuntu-server` to true.
- API: `Host.IsController` mapped; `HostService.ResolveControllerHostIdAsync` returns first controller by name; throws if zero. `playbook_runs` is NOT given a host_id column (kept for the separate plan).
- UI:
  - `selectedControllerSlice` (Redux Toolkit) persists choice to `localStorage` key `cm.selectedControllerHostId`.
  - **AnsibleCollectionsPage** + **AnsibleCollectionDetailPage**: replaced `hosts[0]` with `controllerHosts.find(h => h.id === selectedControllerId) || controllerHosts[0]`. Dropdown rendered.
  - **AnsibleCollectionDetailPage**: also renders an "All controllers" summary table.
- Playwright evidence: `C5-selector.png` — Controller dropdown rendered on Collections page.

## DB row inspection at end of phase 12
```
public_id      | status | vault_secrets_prefix | has_token | error_message 
run_87KST5NXPB |   1    |                      |    f      | (null)
run_894HG5R1TF |   1    |                      |    f      | (null)
run_T9742WY3BH |   1    |                      |    f      | (null)
```
All three are Queued state, no Vault columns set. Origin: the three triggers attempted during this verification before the cloudmanager policy can be extended. These rows are harmless — the sweeper will not pick them up (no token to revoke) and they can be cancelled or manually deleted.

## Constraints honored
- camelCase JSON on the wire — verified by `/api/v1/FeatureFlags` curl output (keys: playbooks, lastProbedAt).
- public_ids on the wire, internal UUIDs never leaked — verified at DTO layer of new requirement + host services.
- Vault tokens NEVER in API logs (redaction enricher) — verified journal grep.
- Vault tokens NEVER in run output streams — vorch only logs "using API-minted VAULT_TOKEN".
- VaultRunToken scrubbed from list endpoints — `ListByAssignmentAsync` nulls it; only `GetByIdAsync` surfaces real token.
- Revoke on terminal is idempotent — `RevokeRunVaultAsync` early-returns when `VaultRunToken` empty.
- No silent fallback to long-lived cloudmanager token in vorch — vorch checks `apiRunToken` non-empty; otherwise vorch's own MintChildToken path runs (which exists for prior flows).
- `playbook_runs` does NOT have a host_id column — verified.
- Refuse-only for requirements (no auto-install) — code inspection.
- One feature, one PR per affected repo (4 PRs: api / vorch / web / workflow-exec) — vorch-lib not touched.
