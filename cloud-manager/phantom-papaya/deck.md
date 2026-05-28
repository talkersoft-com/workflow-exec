# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-web` | (conditional) Edits to `CreateVmPage.tsx` or `services/api.ts` if the bug is on the request side. |
| `cloud-manager-api` | (conditional) Edits to `VirtualMachineController.cs` / its DTO / orchestration publisher if validation, model binding, DB, or AMQP publish is the failure layer. |
| `vorch-service` | (conditional) Edits to `handlers/vm_handlers.go` or `messagesub/messagesub.go` if the Go consumer crashes or fails to surface a real libvirt error. |
| `vorch-lib` | (conditional) Edits to `provision.InitializeHost` or related provision code if the failure is in the libvirt orchestration layer. |
| `workflow-exec` | Always — workflow scaffold + Results/Lessons at ship time. |

The actual set of touched repos is determined in Phase 2 (Diagnose). Most likely a single code repo plus `workflow-exec`. Repos not touched will still ride on the branch but be skipped by `hv_ship`.

## Branch
`giddy-waterbuffalo` (execution; planning was on `phantom-papaya`)

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_init    deck: "cloud-manager"
```
If already provisioned on a prior feature branch with all PRs merged:
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "fix: VM creation error in cloud-manager"
         title:   "fix: VM creation error in cloud-manager"
```

## Operational notes for this workflow
- The workflow runs ON `ubuntu-server.talkersoft.com` — the same host that runs the deployed cloud-manager stack. `localhost:5250` is the live API and `localhost:3000` is the live web app behind nginx at `https://ubuntu-server.talkersoft.com`.
- Passwordless `sudo` is available for `journalctl`, `systemctl`, and `install-*.py`.
- Deployment scripts live in `vm-infra/cloud-manager/cloud-manager-api/scripts/`:
  - API: `scripts/api/install-api-service.py`
  - Web: `scripts/web/install-web-app.py` (run `sudo rm -rf <web>/dist` first to avoid permission errors from prior deploys)
  - Vorch: `scripts/vorch/install-vorch-service.py`
- Database access for verification: `database-toolkit` MCP, infra `clouddb` at localhost:5432.
- RabbitMQ runs as systemd unit `rabbitmq-server`; management UI at port 15672 if needed for queue inspection.
