# Result: workflow-relocation

## Workflow ID
`0001-WORKFLOW-RELOCATION`

## Outcome
SHIPPED

## Branch
`amiable-pancake` (cloud-manager deck) / `warm-leviathan` (hive-deck-pro)

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `hive-deck-pro` | TBD — filled from hv_ship output | — |

## Phase summary

### Phase 0 — Initialize
All 14 cloud-manager repos clean on `amiable-pancake`. Branch already provisioned from prior ship; no hv_init/hv_next required. Branch name recorded in deck.md. Note: tracked `.md` files were missing from the working tree after branch switch — restored with `git checkout HEAD -- .`. Root cause unknown (not skip-worktree); flagged in LESSONS.

### Phase 1 — Struct changes
Added `Repo string \`yaml:"repo"\`` to `WorkflowDef`. Removed `WorkflowFolder string` field from `DeckFile`. Build clean immediately.

### Phase 2 — workflowCmd update
Moved `LoadWorkflows()` call before plan resolution. Replaced `l.DeckFile.WorkflowFolder` lookup with `wf.Repo`. Added org-strip logic (`strings.LastIndex(wfRepoName, "/")`) because `RepoPlan.Repo` stores only the bare repo name while `WorkflowDef.Repo` stores the full `org/repo` ref. Error message: `workflow "X" requires repo "Y" which is not in this deck`.

### Phase 3 — Validation
Added `validateWorkflow()` to `ValidateDeck()`. Walks the deck tree via `collectRepos()` to collect all `org/repo` refs, then checks the workflow's `repo` field against that set. No `resolve` import needed — `collectRepos` is a pure tree walk, avoiding a config→resolve import cycle.

### Phase 4 — Config files
Added `repo: talkersoft-com/execution-workflow` to `~/.hv/workflows.yaml` and repo copy. Removed `workflow_folder:` from `~/.hv/decks/cloud-manager.yaml` and repo copy. All diffs clean, files in sync.
