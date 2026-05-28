# Lessons: workflow-relocation

## Workflow ID
`0001-WORKFLOW-RELOCATION`

## What would have helped

1. **`RepoPlan.Repo` stores bare name, not `org/repo`** — The task file said "find that repo in `plan.Repos`" without flagging that `RepoPlan.Repo` is just the repo name (e.g., `execution-workflow`) while `WorkflowDef.Repo` is the full ref (`talkersoft-com/execution-workflow`). Future tasks touching `resolve.RepoPlan` should document this distinction upfront.

2. **Tracked `.md` files disappear after branch switch** — On `amiable-pancake`, all tracked `.md` files in `execution-workflow` were missing from disk. `git checkout HEAD -- .` restored them. Not skip-worktree (confirmed via `git ls-files -v`). Possibly a git behavior with the `*.md` gitignore rule interacting with branch checkout, or an issue in `hv_next`'s checkout logic. Worth investigating in hive-deck-pro: if `hv_next` does a checkout that skips gitignore-matching tracked files, that's a bug.

3. **`config` cannot import `resolve`** — Adding validation in `validate.go` required walking the deck tree manually rather than calling `resolve.Build`. The task file anticipated this correctly. Document the package dependency direction: `cmd/hv` → `resolve` → `config`; never `config` → `resolve`.

4. **Installed binary is stale** — The `hv` binary in `$GOPATH/bin` was an old version. Testing required `CGO_ENABLED=0 go build -o /tmp/hv-test`. Consider adding a `make install` or noting in the workflow task template to reinstall after code changes.

## Fix files written
None.
