# LESSONS — cloud-manager/nostalgic-jetplane

## What went well
- The dependency surface (#2 `vm_secret_bindings` table, #1.5 `Secret` entity, the blueprint↔binding
  PATCH endpoint) was all present on origin/hive, so Task 0000 cleared with no surprises.
- An existing `VmSecretBindingService` already owned the `vm_secret_bindings` table — the read method
  landed there naturally, mirroring the paradigm rather than inventing one. `SecretBindingResolver`
  already encoded the static-vs-templated path rule, so `ResolveVaultPath` is a faithful path-only
  copy (it never reads Vault, preserving the no-value rule).
- camelCase + public-ids fell out of the existing DTO serialization and `GuidByPublicIdAsync` helpers;
  the absolute-route trick (`[HttpGet("/api/v1/vm/{publicId}/secret-binding")]`) matched how
  `SecretBindingController` defines its blueprint-scoped routes.

## What would have helped (and the one real snag)
- **Templated bindings can't be provisioned without a pre-seeded Vault secret.** The round-trip's
  first provision 500'd because a templated binding resolves to
  `cloudmanager/data/vm/instances/{vm_public_id}/…`, and the resolver reads that Vault path **at
  instantiate** to inject — the path is empty, so it throws `VaultSecretNotFoundException`. The VM id
  isn't known until instantiate, so the path can't be seeded ahead of time. See **Retro/FIX-001**.
  The fix was to drive the round-trip with a **static** binding (a managed `Secret` at a fixed
  `cloudmanager/data/manual/<slug>` path that the resolver can actually read). A test note in Task
  0004 ("use a static binding, or pre-seed the templated Vault path") would have saved one failed VM.
- **The two new MCP tools are not callable in the same session that builds them.** Adding a tool is a
  contract change that only takes effect after an operator `/mcp` reload, so Tests 0002/0003 were
  verified through the live API endpoints the tools wrap (exact path + body) rather than by invoking
  the tools over MCP. Worth stating up front in any MCP-tool task: per-phase "live tool call" is
  blocked until reload; verify the wrapped endpoint instead.
- **`db_execute_sql_query` never returns row data** (counts/rows-affected only). Verifying the actual
  JSON shape and the no-value rule had to go through the HTTP endpoint (curl), not the DB tool. Good
  for the soft-delete toggle (UPDATE … rows-affected), not for reading the row back.

## FIX files written
- `Retro/FIX-001.md` — provision 500 on a templated binding (Vault path not seeded); pivoted the
  round-trip fixture to a static binding. Not a code defect in this workflow.

## Follow-ups / housekeeping
- Live test fixtures (`vm_89PF4XWDCZ`, the failed `vm_PPR04V26XF`, two blueprints, three bindings, one
  secret) remain in the system as evidence; tear down at will (listed in Results/RESULT.md).
- Open question from the PLAN resolved as designed: `resolvedVaultPath` returns the **resolved**
  (substituted) path. For a static binding it is the referenced `Secret.vault_path`; for a templated
  binding it is `path_template` with `{vm_public_id}` substituted.
