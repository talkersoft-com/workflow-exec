# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes

| Repo | What changes |
|------|-------------|
| `database-toolkit` | Wire `resolveVaultEntries` into both config-loader paths in `tools/shared/database-config-loader.ts` (`loadDatabaseConfigWithFallback`, `loadTestDatabaseConfigWithFallback`), rebuild dist |
| `workflow-plans` | Findings docs from Tasks 0002 and 0003 land in `cloud-manager/beaming-messenger/` |
| `workflow-exec` | This workflow folder (orchestration, results, retro) |
| `cloud-manager-api` | Task 0002 conditional fired (playbook `pg-14-jammy` owns postgres setup): `seed/playbooks/pg-{14-jammy,16-noble}.yaml` get listen_addresses + scram pg_hba network config (live playbooks updated in parallel via `cloud_playbook_update`) |

Repos not listed will be on the feature branch but skipped by `hv_ship`.

## Branch

`gloppy-snapper` — the deck is already on this branch from the planning ship + auto-transition (all 18 repos confirmed clean on it).

## Initialize (Task 0000)

```
hv_status  deck: "cloud-manager"
```

Expect all repos on `gloppy-snapper`, all clean. If somehow not on a fresh feature branch:

```
hv_next  deck: "cloud-manager"
```

(then update the Branch field above with the newly generated name).

## Ship

```
hv_ship  deck: "cloud-manager"
         message: "database-toolkit: wire vault-resolver into config loaders; pg-test provisioning + ssh-secret findings"
         title:   "Vault-backed db connections: wire the resolver; provisioning + ssh-secret findings"
```
