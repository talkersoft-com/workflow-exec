# Branches: disksize-disco

This workflow modifies the following repos. **The orchestrator MUST create a feature branch in every repo listed here before starting Task 0001.**

## Repos for this workflow

| Repo path (relative to `~/workspace/`) | Branch | Purpose |
|---|---|---|
| `cloud-manager/vm-infra/cloud-manager/cloud-manager-api` | `disksize-disco` | Entity + migration, DTO, AMQP message, controller |
| `cloud-manager/vm-infra/cloud-manager/cloud-manager-web` | `disksize-disco` | Create VM form field, types, api client |
| `cloud-manager/vm-infra/cloud-manager/cloud-manager-mcp` | `disksize-disco` | `vm_create` MCP tool param |
| `cloud-manager/vm-infra/cloud-manager/vorch-service` | `disksize-disco` | Go handler: qemu-img resize + growpart on provision |
| `cloud-manager/planning/execution-workflow` | `disksize-disco` | This workflow's docs (already on the branch) |

## How to set up (run this BEFORE Task 0001)

```bash
for d in \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-web \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-mcp \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/vorch-service; do
  echo "=== $d ==="
  git -C "$d" fetch origin
  git -C "$d" checkout main && git -C "$d" pull --ff-only
  git -C "$d" checkout -b disksize-disco
done
```

The execution-workflow repo is already on the branch where this file lives.

## What was authorized

The operator authorized branching every repo above. Do NOT branch repos not listed here — if you discover the work needs an additional repo, stop, update this file, and confirm with the operator before branching.

## Notes

- If a repo doesn't end up needing changes after all, leave the branch dormant and delete it before opening a PR.
- Match branch name across repos so reviewers can cross-reference (e.g. `disksize-disco` everywhere).
