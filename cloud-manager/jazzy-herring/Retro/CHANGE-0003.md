# CHANGE-0003 — Porch wires consumers + healthprobe endpoint added

## cmd/porch/main.go
Real entry point written at `vorch-service/cmd/porch/main.go`:
- Loads server config via `vorch-lib` (same loader vorch uses).
- Dials AMQP from `serverCfg.AMQP.{IP,Port,User,Password}`.
- Registers four consumers via `internal/ansiblesub`: playbook-runs, playbook-run-cancel, collection-installs, collection-removes.
- Starts the healthprobe goroutine.
- Wraps stdout/stderr (and optional `/kvm-automator/porch.log` log file when `RUNNING_AS_SERVICE=true`) with a `redactingWriter` that scrubs `hvs\.[A-Za-z0-9_\-]+` → `hvs.<REDACTED>`. This satisfies Phase 5's log-redaction requirement on the porch side (porch handler code never gets the chance to leak a raw token even via fmt or log).
- Blocks on `select {}` at the end so consumers (which run in goroutines and return immediately) keep ticking.

## Healthprobe endpoint verification — FIX needed
The probe was calling `PATCH /api/v1/admin/feature-flags/playbooks` with body `{healthy, last_probed_at}`. The existing `FeatureFlagsController` only had `PATCH /api/v1/FeatureFlags/{key}` accepting `{Enabled: bool}` — that's the operator toggle, NOT a healthprobe writeback.

Fixed by adding a new endpoint to `cloud-manager-api`:

- **Service**: `IFeatureFlagService.UpdateHealthAsync(string key, bool healthy, DateTime lastProbedAt)` + impl in `FeatureFlagService` that sets `flag.Healthy` and `flag.LastProbedAt` and SaveChanges.
- **Controller**: `[HttpPatch("{key}/health")]` action `UpdateHealth` on `FeatureFlagsController`. Accepts `{Healthy: bool, LastProbedAt: DateTime}`. 404 if key not found; 500 on internal error.
- **Probe**: updated to `PATCH /api/v1/FeatureFlags/playbooks/health` with body `{"healthy": ..., "lastProbedAt": "..."}` (camelCase to match the API wire convention).

The original operator-facing `PATCH /api/v1/FeatureFlags/{key}` (enabled toggle) is unchanged.

## Build status
`go build ./...` on vorch-service: exit 0. The matching API change still needs `dotnet build` (run in Phase 6 with deploy).
