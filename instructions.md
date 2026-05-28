# Execution Workflow — Instructions

## What This Repo Is

A structured workspace for AI agent execution. Each workflow lives in its own named folder with its own `init.md` as the agent entry point.

## Creating a New Workflow

### 1. Pick a Silly Name (with at least one real word about the actual project)

Name your workflow folder after what it does — short, lowercase, hyphenated. Keep it descriptive but light.

**The name must contain at least one real word that names the actual project, system, or component being worked on.** Pure whimsy is not acceptable — six months from now someone has to be able to read the folder name and know what it's for. The silly part exists to make the workflow memorable; the real word exists to make it findable.

Examples:
- `ansible-shenanigans` — real: "ansible"; silly: "shenanigans"
- `vault-rumpus` — real: "vault"; silly: "rumpus"
- `entity-brouhaha` — real: "entity"; silly: "brouhaha"
- `playbook-jamboree` — real: "playbook"; silly: "jamboree"

Bad examples: `the-grand-plan`, `widget-factory`, `magic-time` — no real project word.

> **Note on branch names:** The workflow folder name is the canonical label for all files and docs in this workflow. It is **not** the git branch name — hive-deck auto-generates branch names. See Section 3 below.

The name becomes the prefix for every file inside the workflow (e.g. `0001-ENTITY-BROUHAHA-TASK.md`).

### 2. Create the Folder Structure

```
your-workflow-name/
  init.md         ← agent entry point for this workflow
  deck.md         ← DECK MANIFEST: which hive-deck deck to use + repos with code changes
  Orchestrate/    ← high-level plan, task list, /loop directive
  Tasks/          ← atomic implementation steps (Task 0000 = hv_status + hv_init)
  Test/           ← pass/fail test cases per task
  Improvise/      ← failure adaptations, written before retrying
  Results/        ← one result file per workflow run
  SelfImprove/    ← post-run reflections — what tools/MCPs/access would have helped
```

### 3. Branching — always use hive-deck

**Never use raw `git` commands to create branches.** All branch management goes through the hive-deck MCP.

The standard branching sequence at the start of every workflow is:

```
1. hv_status  deck: "<deck>"   ← verify workspace is clean before touching anything
2. hv_init    deck: "<deck>"   ← provisions all repos + creates a feature branch across all of them
```

`hv_init` auto-generates the branch name — **the agent never supplies one**. After `hv_init`, record the generated branch name in the orchestration file for reference.

If the workspace is already provisioned and on a feature branch from a prior shipped workflow, use `hv_next` instead of `hv_init`:

```
1. hv_status  deck: "<deck>"   ← verify all repos are clean and PRs merged
2. hv_next    deck: "<deck>"   ← transition all repos to a fresh feature branch from origin/main
```

### 4. Number Your Files

All files are numbered with a 4-digit prefix: `0001`, `0002`, etc. The number is shared across all files in a single workflow run. A second workflow in the same folder would be `0002-*`.

### 5. Write init.md First

`init.md` is the agent's entry point. It should explain what the workflow is for, what the agent should read before starting, and any context or constraints. Reference `deck.md` instead of `branches.md`.

### 5b. Write deck.md (the deck manifest)

Replaces the old `branches.md`. Specifies:
- Which hive-deck deck to use (e.g. `cloud-manager`)
- Which repos within the deck will have code changes (for human reviewers)
- The generated branch name (filled in after `hv_init` runs)

Task 0000 is always "run `hv_status` then `hv_init` (or `hv_next`) against the deck". Without this step the agent works on whatever branch the workspace happens to be on — which is wrong.

### 6. Start with the Orchestration

Write `Orchestrate/NNNN-YOUR-WORKFLOW-ORCH.md` first. It defines the objective, task list, and the `/loop` directive. Always include Task 0000 as the first unchecked task.

### 7. Follow the Pattern

See `archive/disksize-disco/` for a historical example of the file shapes. Note that it used raw git commands — **do not copy that branching pattern**. Copy the task/test/orchestration file shapes, not the git commands.

### 8. Always Write a Result

Every workflow ends with a result file — success or failure. A `FAILURE` result is not a bad outcome; it is an honest record. Include the auto-generated branch name and any open PR URLs in the result.

### 9. Always Write a SelfImprove Wishlist

After the Result, write one `SelfImprove/NNNN-WORKFLOW-NAME-WISHLIST-NNN.md` capturing what would have made the run better. Be specific — name the missing MCP method, link the doc that should exist, or describe the exact friction. If everything went smoothly, write a one-liner saying so.

### 10. Ship with hive-deck — the workflow isn't done until main has the code

The final phase of every complete workflow is the ship phase. For the deck:

1. Verify: `hv_status deck: "<deck>"` — all repos with changes should be committed and pushed
2. Ship: `hv_ship deck: "<deck>" message: "<commit message>" title: "<PR title>"` — commits any uncommitted changes, pushes, opens PRs, and (if `auto_merge: true` in config) merges them
3. After merge: `hv_next deck: "<deck>"` — transitions all repos to a fresh feature branch ready for the next workflow
4. Update `Results/` with the PR URLs and merge commit SHAs

**Why this matters:** Code on an unmerged branch is invisible to the next workflow. Use `hv_list_pulls deck: "<deck>"` to confirm all PRs are merged before calling `hv_next`.

> **Partial workflows:** Some workflows stop before shipping (awaiting operator review). In that case, do NOT call `hv_ship`. Document the stop point clearly in the Result and note that shipping will happen after review.

---

## The /loop Directive

Paste this into the agent prompt to run a workflow autonomously:

```
/loop Continue executing tasks in Orchestrate/NNNN-YOUR-WORKFLOW-ORCH.md. Start by running hv_status and hv_init (or hv_next) on the deck. After each task, run the corresponding test. If a test fails, write an improvisation and retry. Check off tasks as they complete. Stop when all tasks are complete or a failure cannot be recovered. Write a result when done.
```

---

## Conventions

| Thing | Convention |
|---|---|
| Folder name | `lowercase-hyphenated` (workflow label — NOT the branch name) |
| Branch name | Auto-generated by hive-deck (`hv_init` / `hv_next`) — record after init |
| Deck | Declared in `deck.md`; always `cloud-manager` for this workspace |
| Agent entry point | `init.md` (inside each workflow folder) |
| Deck manifest | `deck.md` (inside each workflow folder) |
| File prefix | `NNNN-WORKFLOW-NAME` |
| Orchestration suffix | `-ORCH.md` |
| Task suffix | `-TASK.md` |
| Test suffix | `-TEST.md` |
| Improvisation suffix | `-IMPROVISE-NNN.md` |
| Result suffix | `-RESULT.md` |
| Wishlist suffix | `-WISHLIST-NNN.md` |
| First task | `Tasks/0000-WORKFLOW-NAME-TASK.md` — always `hv_status` + `hv_init` / `hv_next` |
| Shipping | `hv_ship` — never raw `git push` + `gh pr create` |
| Branch transition | `hv_next` — never raw `git checkout -b` |

## hive-deck MCP Quick Reference

| Operation | MCP call |
|---|---|
| List available decks | `hv_decks` |
| Check workspace state | `hv_status deck: "cloud-manager"` |
| Provision + branch | `hv_init deck: "cloud-manager"` |
| Commit + push + open PRs | `hv_ship deck: "cloud-manager" message: "..." title: "..."` |
| Transition to new branch | `hv_next deck: "cloud-manager"` |
| Pull latest on current branch | `hv_sync deck: "cloud-manager"` |
| List open PRs | `hv_list_pulls deck: "cloud-manager"` |
| Escape stale branch deadlock | `hv_stash` → `hv_next` → `hv_unstash` |
