# Deck: cloud-manager

## Deck name
`cloud-manager`

## Orchestration id (plan codename)
`splendid-mermaid`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-api` | P0: reorder `MarketplaceController.Instantiate` to resolve bindings before VM creation, 422 + failure-event guard; P1: `InstantiateRequest.secretBindingParams`, `SecretBindingResolver.ResolveForBlueprintAsync` uses supplied params |
| `cloud-manager-mcp` | `cloud_marketplace_provision` gains optional `secretBindingParams` passthrough |
| `workflow-exec` | This orchestration folder (runtime) |

**Must NOT change:** vorch (`vorch-lib`, `vorch-service`), web, CLI, DB schema (no migration), and
every other repo in the deck.

## Branch
`ventured-hyena` — execution branch minted by the warm-up `hv_next` on 2026-06-26 (from origin/hive,
16 repos). Task 0000 re-verified and recorded it here; no second branch was created.

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"   # only if clean on a feature branch with all PRs merged
```

## Integrate / ship (final phase, AFTER deploy, BEFORE done)
```
hv_integrate  deck:    "cloud-manager"
              message: "feat: secret-binding provisioning hardening + instantiate-time param resolution (splendid-mermaid)"
              title:   "Secret Binding provisioning hardening + instantiate-time param resolution (splendid-mermaid)"
              body:    "Workflow #4 of Secret Bindings. P0 resolve-before-create + clean 422; P1 secretBindingParams; MCP passthrough."
```
PRs are queued for auto-merge; `hv_integrate` detects the merges and transitions the deck
automatically.
