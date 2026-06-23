# Lessons: Secret Bindings — authoring foundation

## What worked

- **Mirror an existing seam, exactly.** Treating `blueprint_playbooks` as the literal template
  (entity → DbContext config → DTO → AutoMapper profile → service with two-phase position renumber →
  controller → MCP tool pair → web attach section) meant the whole vertical slice fell out with no
  invented paradigm. The two-phase renumber (offset by 1,000,000, then 0..n) was copied verbatim
  because the `(blueprint_id, position)` unique index rejects intermediate one-pass states.
- **Deploy the API early as a bridge.** The MCP (Task 0002) and web (Task 0003) tests both require a
  *live* API carrying the new endpoints, but the workflow's explicit deploy step is Task 0004. The
  migration is purely additive (two new tables, no edits to existing ones), so deploying the new API
  right after Task 0001 was safe for the running old build and unblocked live integration testing for
  every later task. Task 0004's `--target all` re-deploy is idempotent.
- **Validate the migration both directions on the real DB.** `database update <prev>` then back to
  `AddSecretBindings` proved reversibility against live `clouddb` before any code depended on it.

## Gotchas (worth remembering)

- **The live DB password is NOT `appsettings.json`'s `P@ssw0rd!`.** It is a systemd-encrypted
  credential, decrypted at runtime to `/run/credentials/cloud-manager-api.service/DBPASSWD`
  (root-readable). The design-time `CloudManagerDbContextFactory` uses the *wrong* appsettings string,
  so every `dotnet ef ... database update` for a test apply must pass `--connection` explicitly with
  the real credential. (Confirms the standing `dotnet ef targets live clouddb` note.)
- **`Program.cs` validates DB + RabbitMQ env vars at startup**, so even `dotnet ef migrations add`
  (which never connects) fails unless `DBHOST/DBUSER/DBPASSWD/DBPORT/DATABASE/RABBITMQ_*` are exported.
  `RABBITMQ_PASSWORD` is likewise a systemd credential.
- **Run a second API instance for pre-deploy CRUD testing via `API_BIND_URL`.** The codebase
  supports this for exactly this purpose (a P2 retro lesson). Pick a confirmed-free port — `5260` was
  already taken by another local service and the bind failed silently into a generic 404; `5271` was
  free. Always `ss -ltn | grep` the candidate port first.
- **`.cicd/` scripts live in `cloud-manager-mcp`, not in each repo.** Invoke
  `python3 ../cloud-manager-mcp/.cicd/build-cloud-manager.py` from api/web; they resolve sibling repos
  via `git rev-parse --show-toplevel`.
- **`params`/`config` are stored as raw JSON *strings* in `jsonb` columns** (mirroring
  `vars_override`), so on the wire `params` serializes as a JSON-encoded string, e.g. `"[]"` — not a
  bare array. The web + MCP parse it. This matches the existing blueprint-playbook convention; don't
  "fix" it to a typed array without changing the established contract.
- **localhost:3000 has no API proxy** — UI verification must go through
  `https://ubuntu-server.talkersoft.com` (confirms the standing cloud-manager note).

## For workflow #2 (provisioning)

The seams are in place and exercised at the schema level only: `params[].source` (today `"vm"`,
defaulting when a placeholder is named `vm_public_id`), `injection_provider` (default
`"systemd-creds"`), and the join's `cred_name` (e.g. `DBPASSWD`). The follow-up resolves params at
instantiate and injects via cloud-init/systemd-creds — no migration churn needed.
