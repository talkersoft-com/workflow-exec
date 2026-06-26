# Result: Secret Binding provisioning + injection (systemd creds via cloud-init)

**Orchestration:** cloud-manager/primordial-crystalball (workflow #2)
**Execution branch:** `undertone-devilfish` (minted by hv_next from origin/hive, all 16 repos)
**Date:** 2026-06-26
**Outcome:** ✅ Complete — all 7 phases implemented, every TEST file passed, E2E green on a live VM, api deployed, vorch rebuilt + restarted.

## What shipped
At blueprint instantiate, the controller resolves a blueprint's attached Secret Bindings, reads the
secret value(s) from Vault, and injects them into the new VM as **systemd encrypted credentials** via
cloud-init NoCloud user-data. The guest seals each on first boot (`systemd-creds encrypt --with-key=host`)
into `/etc/credstore.encrypted/<credName>` and shreds the tmpfs plaintext. The VM is never a Vault
client. A `vm.vm_secret_bindings` usage row is written on provision success and gates binding deletion.

## Repos changed (PRs)
| Repo | Change | PR |
|------|--------|----|
| `cloud-manager-api` | `ReadKvSecretAsync` (KV v2) + `VaultSecretNotFoundException`; `SecretBindingResolver` (static/templated path → Vault read → `[{credName,b64Value}]`); `SecretCredData` + `Secrets` on `VmCommandData`; `VmSecretBindingService` (idempotent usage rows on provision success); binding-delete usage guard (409) | _(filled by hv_integrate below)_ |
| `vorch-lib` | `SecretCred` on `createvm.VM` (YAML lockstep); `buildUserData`/`injectSecrets` — per-secret `write_files` (base64 → tmpfs 0600 root) + first-boot seal script (`systemd-creds encrypt` + `shred`); `validateCredName` charset guard | _(filled below)_ |
| `vorch-service` | `Secrets` on the create-vm consumer model; threaded into `InitializeHost` | _(filled below)_ |

> PR links are reported in the final hv_integrate step (see chat output / `hv_list_pulls`).

## Phase summary
| Phase | Result | Evidence |
|-------|--------|----------|
| 0000 Setup | ✅ | branch recorded; #1.5 schema confirmed (`marketplace.secrets`, `secret_bindings.secret_id`, `vm.vm_secret_bindings`) via db-toolkit |
| 0001 Vault KV read | ✅ | live read of `cloudmanager/data/manual/t-read` returned field map; 404 → `VaultSecretNotFoundException`; no value logged; api builds |
| 0002 Resolve bindings | ✅ | live runner: static + templated resolve to correct base64; no-bindings→empty; missing secret→typed error; value never logged |
| 0003 Contract lockstep | ✅ | C# serializer emits `secrets/credname/b64value`; Go round-trips C# YAML + Go fixture; legacy (no `secrets`) → empty, no error (committed Go tests) |
| 0004 vorch injection | ✅ | golden tests: write_files (tmpfs 0600) + seal+shred runcmd; value only in `content:`; `../evil` rejected; empty byte-identical; valid YAML |
| 0005 Usage rows | ✅ | live runner: one `vsb_…` row per binding, idempotent; API gate 409 + DB RESTRICT both block delete; clears after usage removed |
| 0006 E2E + ship | ✅ | see below |

## E2E evidence (live VM `vm_NCGEGFCRFH`, Ubuntu 24.04 Noble, systemd 255)
- **Injection (host seed):** `/var/lib/libvirt/images/vm_NCGEGFCRFH/user-data` contained the
  `write_files` entry `/run/cm-seed/DBPASSWD` (0600 root) with the base64 value, plus the seal script
  (`systemd-creds encrypt --with-key=host --name=DBPASSWD - /etc/credstore.encrypted/DBPASSWD` + `shred`).
  The base64 decoded **exactly** to the managed secret value.
- **TC-001 sealed cred:** in-guest `/etc/credstore.encrypted/DBPASSWD` (0644 root, dir 0700);
  `systemd-creds decrypt --name=DBPASSWD` returned the **original** secret (DECRYPT MATCH).
- **TC-002 no plaintext left:** `/run/cm-seed/` empty after first boot (plaintext shredded).
- **TC-003 no value leaked:** secret value (and its base64) appear in **0** api / vorch / cloud-init
  log lines. (auth.log hits were the verification `grep` command's own literal, not provisioning.)
- **TC-004 usage row + gate:** `vm.vm_secret_bindings` row `vsb_CNFJDTSP46 / DBPASSWD / sb_F7AX988PR7`
  written on provision Success; deleting the in-use binding → **HTTP 409** "in use by 1 VM(s)"; a raw
  `DELETE` of the binding → **FK RESTRICT** violation.
- **TC-005 no-binding regression:** plain `jammy` blueprint VM `vm_SATX7HVPSK` → user-data has **0**
  cm-seed lines; write_files = netplan only; runcmd = the base 5 commands (byte-identical to pre-change).

## Build & deploy record
- **Builds:** `python3 .cicd/build-cloud-manager.py --target api` → clean; `vorch-lib` `go build ./...` +
  `go test ./...` → clean; `vorch-service` `go build ./...` → clean.
- **API deploy:** `python3 .cicd/deploy-cloud-manager.py --target api` → published, service swapped &
  restarted, HTTP 200 after 4s; `cloud_health_check` → **Healthy**.
- **vorch redeploy (manual, hypervisor host `ubuntu-server`):** built the binary, then:
  ```
  go build -o /tmp/vorch-build/vorch-service ./cmd/vorch-service      # in vorch-service/vorch-service
  sudo cp -a /kvm-automator/vorch-service /kvm-automator/vorch-service.bak-<UTC>
  sudo install -m 0755 -o root -g root /tmp/vorch-build/vorch-service /kvm-automator/vorch-service
  sudo systemctl restart vorch_service          # → active
  ```
  (`deploy-app.sh` not used — it scp's to ubuntu-server.talkersoft.com, i.e. ssh-to-self, which fails;
  binary installed locally per the known constraint.)

## Cleanup
All E2E artifacts removed: both test VMs destroyed (+ leftover seed dirs purged), usage rows/VM rows
deleted, secret binding + blueprint deleted, managed secret deleted (value purged from Vault), and the
Task 0001 `t-read` probe secret removed. No `zz-cb*` rows remain.

## Notes / follow-ups
- `DestroyCloudHost` removes `<disk>/<id>.qcow2` but the VM dir is `<disk>/<id>/` — the per-VM seed
  directory (holding the base64 secret) lingers after destroy. Pre-existing vorch path bug; with secret
  injection it now means a base64 secret survives teardown until manually cleaned. **Recommend a vorch
  follow-up** to `os.RemoveAll(<disk>/<id>)`. (See `Retro/LESSONS.md`.)
