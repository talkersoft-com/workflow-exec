# Task: Feature flag scaffolding

## Task ID
`0002-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
Ship the feature-flag plumbing so subsequent phases can mark routes `[RequireFeatureFlag("playbooks")]` and the web UI can hide entire sections of the app off a single response.

## Context
- Schema, column types, and behavior come straight from `PLAYBOOK-INTEGRATION-PLAN.md` → "Feature Flag" and "Three states, three behaviors" tables.
- Prefix `ff` goes into `EntityPrefixRegistry.cs` next to the existing entries (`vm`, `snap`, etc.).
- The web app already has a Redux store; the flags map should live in its own slice (`featureFlagsSlice`) fetched once at app load.

## Steps

1. **Entity + migration**
   - Add `FeatureFlag` entity inheriting `Audit` in `CloudManager.Entities/Models/`. Columns: `Key (varchar 64, unique)`, `Enabled (bool)`, `Healthy (bool)`, `LastProbedAt (timestamptz nullable)`, `Description (text)`.
   - Add to `CloudManagerDbContext` and `EntityPrefixRegistry` (`[typeof(FeatureFlag)] = "ff"`).
   - `dotnet ef migrations add AddFeatureFlags --project src/Models/CloudManager.Entities --startup-project src/CloudManager.API` (or use the project's existing `scripts/database/add-migration.py` if it does the same thing).
   - Seed one row in the migration's `Up()`: `key='playbooks'`, `enabled=false`, `healthy=false`, `description='Generic Ansible playbook execution.'`.

2. **Service + read endpoint**
   - `IFeatureFlagService` with `Task<IDictionary<string, FeatureFlagDto>> GetAllAsync()` and `Task<FeatureFlag?> GetByKeyAsync(string key)`.
   - `FeatureFlagsController` with `GET /api/v1/feature-flags` returning the map shape from the plan: `{ "playbooks": { "enabled": true, "healthy": true, "last_probed_at": "..." } }`.

3. **Admin toggle endpoint**
   - `PATCH /api/v1/admin/feature-flags/{key}` body `{ enabled: bool }`. No auth gate for now — note in code comment that auth lands later.

4. **`[RequireFeatureFlag]` attribute**
   - Action filter (`IAsyncActionFilter`) — reads route attribute, resolves `IFeatureFlagService`, returns 404 when `enabled=false`, 503 with `{ message: "feature enabled but unhealthy", flag: "playbooks" }` when `enabled=true && healthy=false`, passes through when both true.
   - Apply *temporarily* to a no-op test endpoint `GET /api/v1/playbook` returning `[]` so the test file can exercise all three states.

5. **Web UI slice + gating helper**
   - `featureFlagsSlice` with `fetchFeatureFlags` thunk + `selectFeatureFlag(key)` selector returning `{enabled, healthy}`.
   - Dispatch `fetchFeatureFlags` in `App.tsx` on mount.
   - Add a `<FeatureGate flagKey="playbooks">` component that renders children only when both enabled+healthy.

6. **Build + deploy**
   - `python3 scripts/api/install-api-service.py` for API.
   - Rebuild + deploy web (existing script if present, otherwise `npm run build && sudo cp -r dist/* /opt/cloud-manager-web/`).

## Acceptance Criteria
- `GET /api/v1/feature-flags` returns the expected map shape including a `playbooks` entry with `enabled=false, healthy=false`.
- `GET /api/v1/playbook` (the temp no-op route) returns 404 with `enabled=false`.
- `PATCH /api/v1/admin/feature-flags/playbooks {enabled: true}` flips it; `GET /api/v1/playbook` then returns 503 (healthy still false).
- Manually flip `healthy=true` in DB → `GET /api/v1/playbook` returns `[]` (200).
- Reset to `enabled=false, healthy=false` before finishing — the rest of the workflow assumes the flag is off.

## Test
`Test/0002-ANSIBLE-SHENANIGANS-TEST.md`

## On Failure
Common pitfalls: forgetting `await _next()` in the action filter, returning 200 instead of 404 because the filter wasn't registered globally, web build cache serving the old bundle. Write an Improvise documenting the actual fix.
