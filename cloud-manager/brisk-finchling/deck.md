# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes

| Repo | What changes |
|------|-------------|
| `cloud-manager-api` | `IFeatureFlagService` + `FeatureFlagService` + `FeatureFlagsController` |

All other repos are on the feature branch but have no changes and will be skipped by hv_ship.

## Branch
`brisk-finchling`

## Initialize (Task 0000)

```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```

(Deck was already provisioned on `hazy-kite` with all PRs merged — use `hv_next`, not `hv_init`.)

## Ship

```
hv_ship  deck: "cloud-manager"
         message: "fix: feature flag health endpoint — wire probe write path"
         title:   "fix: feature flag health endpoint — wire probe write path"
```
