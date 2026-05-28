# Reproduction — VM-creation failure

## Reproduction steps (UI)
1. Navigate to `https://ubuntu-server.talkersoft.com/create`
2. Step 1 (Host): `ubuntu-server` (only option, preselected) → Next
3. Step 2 (Image): `Ubuntu 24.04 LTS (Noble)` → Next
4. Step 3 (Configure): Name `repro-vm-1320`, Description `phantom-papaya reproduction`, Memory 2GB, Disk 10GB → Next
5. Step 4 (Review): Click `Create VM`

## Visible error
- The UI does NOT show a user-facing toast or error on the create page.
- After submit, the user is redirected to `/virtualmachine/vm_28QBP0ZWJ5` (VM detail page).
- The detail page displays **Status: Error** under the status panel.
- IP `10.0.150.56` and MAC `54:54:D0:37:9E:A5` were allocated, but the VM is in an Error state.

## HTTP response
The POST to `/api/v1/virtualmachine/create/host_KW7YKFF3YM` returned **200 OK** — the API is reporting success even though the orchestration ultimately failed.

```
[POST] https://ubuntu-server.talkersoft.com/api/v1/virtualmachine/create/host_KW7YKFF3YM => [200]
```

(Captured in `network.md`.)

## Server-side evidence

### cloud-manager-api journal (api.log)
The API published a create-vm message, then received a callback message with `status: Failed`:

```
May 28 13:20:44 ubuntu-server CloudManager.API[3584661]:       VM creation message sent for repro-vm-1320 with OS=linux, Flavor=ubuntu, Tag=noble, CloudImageUrl=https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
...
May 28 13:20:45 ubuntu-server CloudManager.API[3584661]:       Received message: command: create-vm
May 28 13:20:45 ubuntu-server CloudManager.API[3584661]:       status: Failed
May 28 13:20:45 ubuntu-server CloudManager.API[3584661]:           displayname: repro-vm-1320
May 28 13:20:45 ubuntu-server CloudManager.API[3584661]:           publicid: vm_28QBP0ZWJ5
```

The API then UPDATEd the VM row's `orchestration_status` to a non-success value (visible as "Error" in the UI).

### vorch_service journal (vorch.log)
**Empty** — vorch is configured with `RUNNING_AS_SERVICE=true`, which redirects `log.SetOutput` to `/kvm-automator/provision.log` (see `vorch-service/app.go:16-31`). systemd journal has no consumer logs at all for this service.

### vorch provision.log (provision-tail.log)
The actual failure is here:

```
2026/05/28 13:20:44 Received a message: command: create-vm
... (full message payload) ...
2026/05/28 13:20:44 Creating VM vm_28QBP0ZWJ5 (display="repro-vm-1320") with OS=linux, Flavor=ubuntu, Tag=noble, Version=24.04
2026/05/28 13:20:44 Cloud image URL provided: https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
2026/05/28 13:20:44 Base image already exists: /var/lib/libvirt/images/linux/ubuntu/noble-server-cloudimg-amd64.img
2026/05/28 13:20:44 Generating SSH key pair for VM vm_28QBP0ZWJ5 (type: ed25519)
2026/05/28 13:20:45 Failed to store SSH key in Vault for VM vm_28QBP0ZWJ5: vault PUT cloudmanager/data/vm/instances/vm_28QBP0ZWJ5 returned 403: {"errors":["permission denied"]}
2026/05/28 13:20:45 Could not create cloud host: vault key storage failed for VM vm_28QBP0ZWJ5: vault PUT cloudmanager/data/vm/instances/vm_28QBP0ZWJ5 returned 403: {"errors":["permission denied"]}
2026/05/28 13:20:45 Successfully published success message
```

Note two anomalies:
1. Vorch logs `"Successfully published success message"` immediately AFTER logging the cloud-host creation failure. That's contradictory — vorch is publishing a "success" envelope when the underlying operation failed.
2. The API received and recorded the message as `status: Failed`, so something between vorch's "success" log and the message wire is actually marking it Failed — OR the log statement is misleading (vorch is calling it a "success" because the message _publish_ succeeded, not because the operation succeeded).

### Pattern: this fails on every VM create
The same exact error appears on every VM created in the last several hours (in this provision.log tail):
- `vm_YDMBA4620Q` (display `TEST-POST-ANSIBLE`) at `2026/05/28 11:40:43` — same 403 from Vault
- `vm_9M3BQ5TG98` (display `TEST123`) at `2026/05/28 12:18:22` — same 403 from Vault
- `vm_28QBP0ZWJ5` (display `repro-vm-1320`) at `2026/05/28 13:20:45` — same 403 from Vault

This is **not** an intermittent failure. **Every** VM creation is currently broken.

### Vault config in use
From `/kvm-automator/config-server.yaml`:
```yaml
vault:
  address: https://ubuntu-server.talkersoft.com:8200
  keyType: ed25519
  sshSecretPath: cloudmanager/vm/instances
  token: hvs.CAESI…
```
The configured path is `cloudmanager/vm/instances` but the error message names `cloudmanager/data/vm/instances/<vm-id>` — the `data/` infix is a Vault KV-v2 convention, so this is the same logical path. The 403 is a permission/policy issue on the token, not a path-mismatch typo.

## One-paragraph hypothesis

The failure layer is **vorch-service** (which calls vorch-lib's vault client). Vorch generates an SSH key pair for each new VM and PUTs it into Vault under `cloudmanager/data/vm/instances/<vm_public_id>`. Vault returns `403 permission denied`, so vorch aborts `provision.InitializeHost` with `vault key storage failed`. This was recently added work (the user's summary notes vorch-service PR #7 promoted `hashicorp/vault/api` to a direct require), so the token's Vault policy almost certainly never gained `create`/`update` permission on the `cloudmanager/data/vm/instances/*` path. The minimal fix is most likely in the Vault policy / token provisioning step (operational) rather than in code — but there is ALSO a real code issue worth fixing: vorch logs `"Successfully published success message"` immediately after a failed cloud-host creation, which is misleading. Diagnosis (Phase 2) should confirm whether the fix lives in the vorch install / bootstrap script (Vault policy bootstrap), the vorch-lib vault client (better error path / clearer log message), or both.

## Artifact files in this folder
- `repro-start.txt` — reproduction window start time
- `screenshot-error.png` — full-page screenshot of VM detail page showing Status: Error
- `network.md` — captured browser network requests filtered to `virtualmachine`
- `console.txt` — browser console (no errors — bug is server-side)
- `api.log` — cloud-manager-api journal since repro-start (1134 lines)
- `vorch.log` — vorch_service journal since repro-start (empty — vorch writes to provision.log)
- `rmq.log` — rabbitmq-server journal since repro-start (empty — RMQ is healthy)
- `provision-tail.log` — last ~50 lines of `/kvm-automator/provision.log` showing the Vault 403 errors

## Service status during reproduction
All four services were `active` throughout:
- `cloud-manager-api` — active
- `vorch_service` — active (running since `2026-05-28 10:20:09 EDT`)
- `cloud-manager-web` — active
- `rabbitmq-server` — active
