# Result: hive-deck-pro/gritty-doberman

## Outcome
SHIPPED

## Branch
`wispy-paprika` (execution branch; gritty-doberman was the planning/workflow branch)

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `hive-deck-pro` | TBD | merged |
| `workflow-configuration` | TBD | merged |
| `workflow-exec` | TBD | merged |

## Phase summary

### Phase 0 — Verify workspace
All 5 repos clean on `wispy-paprika`. No issues.

### Phase 1 — Config & resolve
- Removed `DefaultTargetBranch`, `TargetBranches`, `DeprecatedDefaultBranch`, `DeprecatedBranches` from `Setup` struct
- Removed validation check requiring `default_target_branch`
- Added `Branch string \`yaml:"branch"\`` to `DeckFile`
- Added `Branch string` to `Plan`; set from `l.DeckFile.Branch` in `Build()`
- Replaced `branchFor(repo, l.Setup)` calls in `collectRepoRefs`/`collectModuleRefs` with `l.DeckFile.Branch`
- Deleted `BranchFor` and `branchFor` functions from resolve package

### Phase 2 — provision & init
- Added `git.EnsureRemoteBranch` to `internal/git/git.go`
- `ops/init.go`: after provision, pushes `plan.Branch` to remote for each repo if non-empty

### Phase 3 — branch & next
- `branch.createAll`: now takes `deckBranch string` instead of `setup config.Setup`
- Uses `deckBranch` (falling back to git-detected `defaultBranch` when empty) as the `remoteRef` base
- Both `Run` and `CreateAll` pass `l.DeckFile.Branch`

### Phase 4 — ship
- `repoMeta.defaultBranch` renamed to `targetBranch`
- Target branch sourced from `plan.Branch` (falling back to `defaultBranch(dir)` when empty)
- `EnsureRemoteBranch` called before `commitsAhead` check so the ref resolves
- Pre-flight check updated to use `targetBranch`

### Phase 5 — YAML updates
- Removed `default_target_branch` and `target_branches` blocks from `config.yaml`
- Added `branch: hive` to 8 active deck YAMLs: cloud-manager, hive-deck-pro, hive-deck, tooling, vm-services, personal-apps, aws, ai
- archive, samples, cloud-manager-inactive left without branch field (fall back to git remote default)

### Phase 6 — Cleanup & build
- `go build ./...` clean
- `make install` succeeded
- `hv list decks` verified working
