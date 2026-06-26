# Lessons: primordial-crystalball (Secret Binding provisioning + injection)

## What went well
- **Layered, independently-verified phases.** Each phase was provable on its own — live Vault read,
  resolver runner against live DB+Vault, Go round-trip from the *actual* C# serializer output, golden
  user-data render, usage-row runner — so the final live E2E was a confirmation, not a debugging session.
- **Reusing the contract type across the boundary.** `createvm.SecretCred` is the single Go type used by
  the wire struct, `InitializeHost`, and the vorch-service consumer model. No parallel duplicate types,
  no conversion glue. On the C# side `SecretCredData` is shared by the resolver and `VmCommandData`.
- **No-op-when-empty by construction.** `buildUserData` returns the base template *verbatim* when there
  are no secrets (early return), so the backward-compatible path is byte-identical by design, not by
  careful string surgery. The golden test pins it.

## Things that bit / would do differently
- **EF transitive assemblies in throwaway runners.** A standalone runner referencing only
  `CloudManager.Data.Services` failed at runtime with `Could not load … EntityFrameworkCore.Relational`.
  Fix: add explicit `Microsoft.EntityFrameworkCore.Relational` + `Npgsql.EntityFrameworkCore.PostgreSQL`
  package refs to the runner so the provider assemblies land in its output.
- **Client-generated FK guids.** Setting `binding.SecretId = secret.Id` *before* `SaveChanges` captured
  `Guid.Empty` (id is DB/interceptor-assigned). Save the parent first, then reference its id.
- **Soft-delete query filters mask readbacks.** Verifying a soft-delete by re-querying the row returns
  null (global `DeletedAt == null` filter) — looks like failure. Assert on "no exception thrown" or use
  `IgnoreQueryFilters()`.
- **Secrets in the test harness leak into auth.log.** Running `sudo grep "<secret>" …` / `EXPECTED=<secret>`
  put the literal into `/var/log/auth.log` and the journal — a self-inflicted "leak" unrelated to the
  feature. Compare hashes or pass values via stdin/env files, not as command-line literals.

## Design decisions worth carrying forward
- **base64 in `write_files` content, decoded by the seal script** (not cloud-init `encoding: b64`). Keeps
  the value a single YAML-safe token, and keeps the *plaintext* out of `runcmd` args entirely — the seal
  script only ever names file paths and validated cred names.
- **credName charset guard (`^[A-Za-z0-9_-]{1,128}$`) mirrors the publicId guardrail.** `../evil`, spaces,
  and shell metachars are rejected before the name touches a path or `--name=`.
- **Gate enforced at two levels.** The DB FK is `ON DELETE RESTRICT` (backstop for any hard delete), but
  binding deletion is a *soft* delete, so the constraint never fires through the API. The API-facing gate
  is a code pre-check (usage count → 409), added here. Both were verified in E2E.

## Follow-ups (out of scope here)
- **vorch destroy leaves the per-VM seed dir.** `DestroyCloudHost` removes `<disk>/<id>.qcow2`, but
  `InitializeHost` writes the VM under `<disk>/<id>/` (qcow2 + `user-data` + `meta-data`). The directory —
  now containing the base64 secret seed — survives teardown. Recommend `os.RemoveAll(<disk>/<id>)` in a
  vorch follow-up. (Cleaned manually this run.)
- **Residual seed at rest.** As the plan documented, the host NoCloud seed holds the base64 value until
  first boot completes; combined with the lingering-dir bug above it can persist longer. A follow-up could
  delete the seed artifact once the domain reports first-boot completion.
- **Field selection convention.** Resolver picks `config.field` → lone field → `password`. Multi-field →
  multi-cred mapping (one binding → several creds) is deferred; revisit if a binding needs to inject more
  than one field from a single secret.
