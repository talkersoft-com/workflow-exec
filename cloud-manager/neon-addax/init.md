# cloud-manager-api — implement ansible REST surface

## What this workflow does
Implements the full REST API for ansible playbook + role management in the
`cloud-manager-api` .NET project, as specified by
`cloud-manager/planning/planning/cloud-manager-ansible/API-PLAN.md`. Lands one
schema migration (`AddAnsibleConstraintsAndTargetTimestamps`) and then walks
through the 9 phases of API-PLAN — AnsibleRole + RoleFile CRUD, revision
write-side, Playbook revisions, PlaybookGlobalRoleRef + argument-specs hook,
playbook-local roles, PlaybookRunTarget reads, materialization endpoint,
multi-VM run trigger, run cancellation — adding DTOs, services, AutoMapper
profiles, controllers, and DI registrations along the way.

End state: every endpoint listed in API-PLAN.md is callable, all gated by
`[RequireFeatureFlag("playbooks")]`, with `dotnet build` clean and at least
a smoke test per controller.

## Read before starting
- `deck.md` — repos in scope (`cloud-manager-api` for the implementation,
  `execution-workflow` for Results+Lessons); hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `../../../planning/cloud-manager-ansible/API-PLAN.md` — the spec (just merged in planning PR #3)
- `../../../planning/cloud-manager-ansible/DATA-MODEL-DELTAS.md` — the migration that Task 0001 must land
- `../../../planning/DATA-MODEL.md` — current ER diagram (authoritative for what's deployed)
- `../../../planning/PLAYBOOK-INTEGRATION-PLAN.md` — feature-flag and Vault-path conventions (still authoritative)

## Pre-shipped facts the implementation must respect
- All 13 entity classes exist at `cloud-manager-api/src/Models/CloudManager.Entities/Models/`. **Do not redefine entities.** Wire missing pieces (DbSet registrations, model config) only where needed.
- 6 EF migrations through `AddPlaybookManagement` are applied to `clouddb`. The new migration in Task 0001 is additive.
- `EntityPrefixRegistry` already has prefixes for every entity in scope (`ansr`, `rf`, `rfrev`, `pbrev`, `pgr`, `plr`, `plrf`, `plrfr`, `prt`). Reuse them; do not invent new ones.
- `ArgumentSpecsTranslator` exists at `src/Services/CloudManager.Data.Services/Playbooks/ArgumentSpecsTranslator.cs` and is unused. Wire it in at Task 0005 (Phase 4 of API-PLAN); do not rewrite it.
- `[RequireFeatureFlag("playbooks")]` attribute exists and is used by `PlaybookController`, `PlaybookAssignmentController`, `RunController`. Every new controller this workflow adds must use it.
- `IPlaybookRunPublisher` exists at `src/Services/CloudManager.AMQP.Publisher/` and publishes to `playbook-runs`. Reuse for the multi-VM trigger; add a new `IPlaybookRunCancelPublisher` for `playbook-run-cancel`.
- Existing controllers (`PlaybookController`, `PlaybookAssignmentController`, `RunController`, `FeatureFlagsController`) follow the patterns new controllers should mirror: try/catch with `_logger.LogError`, `{ Message: ... }` error body, 400/404/409/500 status codes.

## Constraints
- No new public_id prefixes — reuse `EntityPrefixRegistry` entries.
- Every new controller carries `[RequireFeatureFlag("playbooks")]`.
- DTOs surface `public_id` as `publicId` (camelCase on the wire); never expose internal `id` (Guid).
- File-path validation reused across `RoleFile` and `PlaybookRoleFile` (extract to a shared validator): reject `..`, leading `/` or `~`, backslashes, content > 1 MiB.
- Service-layer hooks (revision creation, argument-specs re-derive) must run inside the same EF transaction as the underlying write.
- Every new endpoint documented via XML doc comments so Swashbuckle picks them up.
- No git-backed playbook fields anywhere in new code (no `git_url`, `git_ref`, `playbook_path`, `last_refreshed_at`).
- New `IPlaybookRunCancelPublisher` must publish to a queue named exactly `playbook-run-cancel` — `vorch-service` will subscribe by the same name in a sibling workflow.
- After every implementation task: run `dotnet build` and confirm zero errors. Run `dotnet test` if a test project exists.
- Per-user RBAC is OUT OF SCOPE — leave routes flag-gated only.
