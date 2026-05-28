# Improvise: Phase 0002 route + admin-PATCH divergences

## Phase
`0002-ANSIBLE-SHENANIGANS`

## What the plan said
- Routes: `/api/v1/feature-flags` (kebab) and `/api/v1/playbook` (singular).
- TC-003 said "PATCH /admin/feature-flags/playbooks {enabled: true}".

## What actually happened
- ASP.NET attribute routing on `FeatureFlagsController` auto-resolves to `/api/v1/featureflags` (no hyphen). The `PlaybookController` becomes `/api/v1/playbook`, but list lives at `/api/v1/playbook/list` to match the existing convention (HostController, VirtualMachineController).
- There is no separate `/admin/` route; the PATCH endpoint is just `PATCH /api/v1/featureflags/{key}`. Auth/admin gating is deferred — there is no auth layer in the project today.

## Decision
- Kept attribute routing default (camelCase / no kebab) — consistent with every other controller in the repo. Changing the convention is out of scope for this phase.
- Single PATCH route (no `/admin/` prefix) — matches the rest of the API and avoids inventing an auth concept that doesn't exist yet.

## Verification
All 6 test cases (TC-001 through TC-006) passed against the actual routes:
```
GET  /api/v1/featureflags                → 200 + dict
PATCH /api/v1/featureflags/playbooks     → 200 + updated row
GET  /api/v1/playbook/list (disabled)    → 404
GET  /api/v1/playbook/list (unhealthy)   → 503
GET  /api/v1/playbook/list (healthy)     → 200 []
PATCH /api/v1/featureflags/doesnotexist  → 404
```

## Followup
If we later add admin auth, gate the PATCH endpoint with whatever attribute we land on. No change to clients beyond that.
