# Lessons — wheezy-cottonmouth (Secret Manager)

## What went well
- **Mirroring the `secret_bindings` code paths** made the API phase fast and low-risk: entity →
  DbContext config → prefix registry → DTO → AutoMapper profile → service → controller → DI, one
  established slot at a time. The fluent `HasCheckConstraint` + `HasOne/WithMany/OnDelete(Restrict)`
  config let `dotnet ef migrations add` generate a clean, reversible migration **and** keep the model
  snapshot in sync — no hand-authored migration drift.
- **DB-first state machine** is genuinely robust: every create INSERTs `Pending` before the Vault
  write, so a Vault failure leaves a `Failed` row, never an unknown orphan. Leaf-delete destroys the
  Vault key only after the 409 guard and the RESTRICT FK both pass.
- **Pre-deploy CRUD on a transient API instance + live DB/Vault** (the prior workflow's pattern)
  proved the whole state machine and path-lock before any deploy.

## What cost time (and the fix)
- **The live `clouddb` password is a systemd-encrypted credential, not appsettings.** appsettings.json
  carries the `P@ssw0rd!` placeholder; the real value is `LoadCredentialEncrypted=DBPASSWD` decrypted
  to `/run/credentials/cloud-manager-api.service/DBPASSWD` (root-readable). `psql`, the db-toolkit, and
  even `dotnet ef --connection "...P@ssw0rd!..."` all fail auth; only the cred value works. This is
  already captured in the `dotnet-ef-targets-live-clouddb` memory — consult it first next time.
- **The second API instance won't stand up cleanly next to the live one** because of the RabbitMQ
  consumer (`ConsumerService` opens an AMQP connection in its constructor; `AMQP:user/hostIp` bind from
  the appsettings section, not the `RABBITMQ_*` env vars, and an unhandled `ACCESS_REFUSED` kills the
  process). Recorded as **FIX-001**. The resolution that actually worked — and is the more faithful
  test anyway — was to **bring the Task 0004 API deploy forward** (the change is backward compatible
  and the migration was already applied) and exercise the MCP tools against the real `:5250` service
  under systemd, which has the correct credential wiring. **Next time: deploy the API early and test
  the MCP/round-trip against the deployed service rather than fighting a throwaway second instance.**
- The sandbox terminates long-running **foreground** network-binding commands (exit 144). Launch such
  test servers detached and poll separately.

## Gotchas worth remembering
- KV v2 destroy = DELETE on the **metadata** path (`/data/` → `/metadata/`), which removes all versions;
  the stored `vault_path` is the data path, so swap the segment for the hard-delete.
- `z.record(...)` in this MCP's zod needs an explicit key schema: `z.record(z.string(), z.string())`.
- Web binding↔secret reference: keep the Guid→public_id resolution in the service (never auto-map the
  `SecretId` Guid through AutoMapper) so the wire stays public-id-only; the modal sends `secretId` and
  derives the read-only path from the selected Secret.
- Soft-delete + a filtered unique index on `vault_path` (`WHERE deleted_at IS NULL`) lets a slug be
  reused after a secret is deleted — keep the uniqueness filtered, matching `secret_bindings_name_key`.
