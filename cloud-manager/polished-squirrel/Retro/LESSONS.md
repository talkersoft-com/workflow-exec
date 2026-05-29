# Lessons: cloud-manager/polished-squirrel

## What went smoothly
- `contract.go` already had no `continue_transaction` — the prior linter revert actually saved work here.
- `ShipConfig.UnmarshalYAML` uses a raw map decode so adding `OpenBrowser` required zero parser changes — just the struct field and YAML tag.

## What would have helped
- The `configure-hv-home.sh` path bug (still pointing at `cloud-manager/config`) was noted but out of scope for this workflow — a dedicated task would prevent it recurring across sessions.
- `~/.hv/config.yaml` must be manually synced from `workflow-configuration/config.yaml` — no automation exists for this yet.

## Fix files written
None.
