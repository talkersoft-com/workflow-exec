# Improvise: Phase 0006 worker bring-up

## Phase
`0006-ANSIBLE-SHENANIGANS`

## Issues encountered (in firing order)

### 1. PATH didn't include venv bin → ansible-runner rc=127
ansible-runner shells out to `ansible-playbook`. The systemd unit launched python from the venv, but its `bin/` wasn't on PATH inside the worker process, so subprocess discovery failed with `command not found`. Fixed in the bash wrapper (`cloud-manager-worker`): `export PATH="$VENV_BIN:$PATH"` before exec.

### 2. ProtectHome=read-only blocked `~/.ansible/tmp` → rc=5
Ansible insists on writing to `~/.ansible/tmp/ansible-local-XXX/`. With `ProtectHome=read-only` in the systemd sandbox, that errored with `Errno 30 Read-only file system`. Fixed by adding three env vars to `/etc/cloud-manager/worker.env`:
```
ANSIBLE_LOCAL_TEMP=/tmp/.ansible-local
ANSIBLE_REMOTE_TEMP=/tmp/.ansible-remote
HOME=/tmp/.cloudmanager-home
```
And added a startup loop in `main()` that mkdirs `$HOME/.ansible/tmp` + the two ansible temp dirs (PrivateTmp gives a fresh /tmp each start, so they need recreating).

### 3. `vm.virtual_machines.ip_address` doesn't exist
My worker SQL joined `v.ip_address` directly off the VM table. Reality: IPs live in `network.addresses` keyed by `virtual_machine_id`. Fixed with a correlated subquery: `(SELECT n.ip_address FROM network.addresses n WHERE n.virtual_machine_id = v.id LIMIT 1)`.

### 4. Hermetic test repo had no playable entrypoint
The Phase 0004 hermetic repo at `/var/lib/cloud-manager/test-playbook` only had `tasks/main.yml` (a tasks file, not a play). When the worker invoked `ansible-playbook tasks/main.yml`, ansible failed with `'ansible.builtin.debug' is not a valid attribute for a Play`. Added a top-level `site.yml`:
```yaml
- hosts: all
  gather_facts: yes
  tasks:
    - debug: { msg: "hello from {{ inventory_hostname }} ..." }
    - copy: { dest: /tmp/playbook-marker.txt, content: "applied at {{ ansible_date_time.iso8601 }} ..." }
```
Updated `vm.playbooks.playbook_path='site.yml'` for the existing row.

### 5. Git "dubious ownership" on shared test repo
After moving the test repo into `/var/lib/cloud-manager/test-playbook` (cloudmanager-owned) but running `git commit` as root, git refused due to "dubious ownership". Added `safe.directory` entries for both users.

## Verification
All 7 TCs pass. First successful end-to-end:
- `run_4E0KRMM9DR` → Succeeded in ~4s, ok=3 changed=1 on `vm_ZNCMJE89SR`.
- Marker file `/tmp/playbook-marker.txt` confirmed on the VM via direct SSH read.
- Second run `run_NNS9FSJQH6` re-confirmed all 7 TCs in one shot.

## Followup
- The hermetic test repo's `site.yml` is a smoke playbook only. The actual postgres install for Phase 0010 will need either a real geerlingguy fork with `meta/argument_specs.yml` added, or a different role that already has argument_specs.
- Token at `/etc/cloud-manager/worker.env` is the long-lived operational token I was given. Phase 0007 (scoped per-run tokens) will replace this with child-token minting.
- ProtectHome=read-only kept; the env-var workaround is cleaner than poking holes in the sandbox.

## State
- Flag remains (true, true) so the smoke playbook can run again from the UI.
- VM `vm_ZNCMJE89SR` not torn down per user's standing instruction.
