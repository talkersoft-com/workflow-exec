# LESSONS — cloud-manager/thriving-framehouse

## What would have helped

1. **The resolved path is computed at provision-request time, but the usage row is written from the
   async provision ACK — and the ack does not carry the path back.**
   The plan said "thread the resolved path through to the site where the `VmSecretBinding` usage
   row is inserted." That site (`VmSecretBindingService.RecordUsageForVmAsync`) runs in the AMQP
   consumer with only the VM id; the vorch round-trip drops the resolver output. So the path had to
   be persisted at **provision time** in `MarketplaceController` (the only place it's in scope),
   passing a `credName→path` map into an enhanced `RecordUsageForVmAsync`. The consumer's existing
   call stays as an idempotent null-path fallback. A one-line "the insert happens in the consumer,
   not the controller" note in the plan would have saved a full trace of the provision pipeline.

2. **Keep the resolved path off the vorch message.** `SecretCredData` is serialized to YAML and
   sent to vorch in "lockstep" with its `createvm.VM.Secrets`; `CloudManager.Messages` has no
   YamlDotNet dependency, so a `[YamlIgnore]` couldn't cleanly hide an added field, and adding a
   serialized `vaultpath` risked a strict-deserialize break on vorch (out of scope: "no vorch
   changes"). Surfacing the path via a new service-layer type (`ResolvedSecretBinding`) and mapping
   back to a clean `SecretCredData` for the message kept the contract untouched.

3. **The MCP guard fix can't be self-verified live — the running server must be `/mcp`-reloaded.**
   A build updates `dist/` but not the running MCP process, so the camelCase guard fix is invisible
   until the operator reloads. This is the same gotcha already recorded for new MCP tools. Mitigated
   by (a) unit-testing the guard logic standalone (6 cases), (b) probing the live MCP to confirm it
   still throws the old error (bug reproduced), and (c) curling the deployed API directly to prove
   it accepts the fixed request shape (422 at resolution, not a parse rejection). The fully-green
   live "through-the-MCP" E2E (Pass A + B) remains the operator's post-reload step.

4. **The documented DB connection string carried a stale password.** `P@ssw0rd!` is the port-5433
   `clouddb_local` dev password; the live `clouddb` on 5432 uses the password in the hive db-toolkit
   config (`~/.hv/toolkit/db/config/database.json`). `db_test_connection` is the fastest way to
   confirm the real credential before `dotnet ef database update`. Captured in `FIX-001.md`.

## FIX files written
- `FIX-001.md` — migration `dotnet ef database update` failed `28P01 password authentication
  failed`; root cause was the stale dev password in the documented connection string; fixed by
  using the live `clouddb` password from the db-toolkit config.

## What went smoothly
- The migration is purely additive + nullable, so the live apply was a one-shot with a clean
  reversible `Up`/`Down`; old rows degrade gracefully via the recompute fallback.
- Interface changes had a tiny blast radius (one resolver, one caller, one consumer call site),
  so the return-type change and the optional `resolvedPaths` param were low-risk and compiled clean.
