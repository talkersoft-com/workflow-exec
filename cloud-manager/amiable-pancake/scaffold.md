## Step 0 — Verify deck is clean

Before creating any files or folders, run `hv_status` on the deck:

```
hv_status  deck: "cloud-manager"
```

If **any repo is dirty** (uncommitted changes, unpushed commits, stash entries, etc.), **stop immediately**. Do not create the workflow folder structure. Tell the user which repos are dirty and ask them to clean up before proceeding.

Only continue when `hv_status` reports all repos clean.

## Step 1 — Name the workflow

Choose a `lowercase-hyphenated` name. It must contain at least one real word naming the project, system, or component being worked on, plus at least one other word for memorability.

Good: `entity-brouhaha`, `playbook-jamboree`, `ansible-shenanigans`
Bad: `the-grand-plan`, `widget-factory` — no real project word

This name becomes the workflow folder and the uppercase slug embedded in every filename.

## Step 2 — Determine the run number (NNNN)

Check `/Users/talker/workspace/cloud-manager/planning/execution-workflow/` for existing workflow folders. If none exist, use `0001`. If a prior run exists (e.g. files prefixed `0001-`), use `0002`. The number is shared across every file in a single run.

- `SLUG` = `NNNN-WORKFLOW-NAME` in uppercase with hyphens (e.g. `0001-ENTITY-BROUHAHA`)

## Step 3 — Create the folder structure

Create the workflow folder inside `/Users/talker/workspace/cloud-manager/planning/execution-workflow/`:

```
/Users/talker/workspace/cloud-manager/planning/execution-workflow/<workflow-name>/
  init.md
  deck.md
  Orchestrate/
  Tasks/
  Test/
  Retro/
  Results/
```

`Retro/`, `Results/` start empty — the AI populates them at runtime.

## Step 4 — Write `init.md`

```markdown
# <Workflow Name>

## What this workflow does
<One paragraph: what problem is being solved and what the end state looks like.>

## Read before starting
- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Orchestrate/<SLUG>-ORCH.md` — full task list and /loop directive
- `../../common/toolkit/hive-deck.md` — hv MCP call patterns
- <any design docs, data models, or specs relevant to this workflow>

## Constraints
<Any hard rules the AI must not violate — e.g. "do not modify the public schema",
"migrations must be reversible", "no breaking API changes". Omit section if none.>
```

## Step 5 — Write `deck.md`

```markdown
# Deck: <workflow-name>

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `<repo-name>` | <brief description> |

Repos not listed will be on the feature branch but skipped by hv_ship.

## Branch
`amiable-pancake`

## Initialize (Task 0000)
`‌``
hv_status  deck: "cloud-manager"
hv_init    deck: "cloud-manager"
`‌``
If already provisioned on a prior feature branch with all PRs merged:
`‌``
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
`‌``

## Ship
`‌``
hv_ship  deck: "cloud-manager"
         message: "<commit message>"
         title:   "<PR title>"
`‌``
```

## Step 6 — Write `Orchestrate/<SLUG>-ORCH.md`

```markdown
# Orchestration: <workflow-name>

## Workflow ID
`<SLUG>`

## Objective
<One paragraph describing what done looks like.>

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../common/toolkit/hive-deck.md`
- <additional design docs>

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/<SLUG>-0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next
- [ ] `Tasks/<SLUG>-0001-TASK.md` — **Phase 1**: <description>
- [ ] `Tasks/<SLUG>-0002-TASK.md` — **Phase 2**: <description>
...
- [ ] `Tasks/<SLUG>-NNNN-TASK.md` — **Final phase**: write Results + Retro/LESSONS, then hv_ship

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file
   d. On failure: write `Retro/<SLUG>-FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## Autonomous execution
```
/loop Continue executing tasks in Orchestrate/<SLUG>-ORCH.md. Start by running
hv_status then hv_init (or hv_next) per deck.md. For each unchecked task: read
the task file, do the work, run the matching Test file. On failure write
Retro/<SLUG>-FIX-NNN.md and retry. On pass check the box. When all boxes are
checked write Results/<SLUG>-RESULT.md and Retro/<SLUG>-LESSONS.md — both BEFORE
calling hv_ship — then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/<SLUG>-RESULT.md`
- `Retro/<SLUG>-LESSONS.md`
```

## Step 7 — Write `Tasks/<SLUG>-0000-TASK.md` (always identical)

```markdown
# Task 0000 — Initialize Workspace

## Goal
Verify the workspace is clean and create the feature branch across all repos in the deck.

## Steps
1. Run `hv_status` on the deck (see `../deck.md` for the exact call)
2. If all repos are clean and on the default branch → run `hv_init`
3. If all repos are clean and on a feature branch with all PRs merged → run `hv_next`
4. If any repo is dirty → STOP. Investigate before proceeding.
   - Exception: dirty repo with a merged PR → hv_stash → hv_next → hv_unstash
5. Run `hv_status` again to confirm all repos are on the new feature branch, all clean
6. Record the generated branch name in `../deck.md`

## Definition of done
All repos on the new feature branch, all clean. Branch name recorded in deck.md.
```

## Step 8 — Write `Test/<SLUG>-0000-TEST.md` (always identical)

```markdown
# Test 0000 — Workspace Initialized

## TC-001 — All repos on feature branch
Run `hv_status` on the deck.
Pass: every repo shows the same feature branch name, no repo is on the default branch.
Fail: any repo is on the default branch or a different branch.

## TC-002 — All repos clean
Pass: hv_status shows no dirty repos.
Fail: any repo shows uncommitted changes, unpushed commits, or stash entries.

## TC-003 — Branch name recorded
Open `../deck.md`.
Pass: the Branch field is filled in with the generated branch name (not "TBD").
Fail: Branch field still says TBD or is missing.
```

## Step 9 — Write `Tasks/<SLUG>-NNNN-TASK.md` for each implementation phase

```markdown
# Task <NNNN> — <Phase Name>

## Goal
<One sentence: what this task produces.>

## Target repo(s)
<repo-name>

## Steps
1. <step>
2. <step>
...

## Definition of done
<Concrete, verifiable end state. Matches what the test file will check.>
```

## Step 10 — Write `Test/<SLUG>-NNNN-TEST.md` for each task

```markdown
# Test <NNNN> — <Phase Name>

## TC-001 — <what is verified>
<Command or action to run.>
Pass: <exact expected output or condition.>
Fail: <what a failure looks like.>

## TC-002 — <what is verified>
...
```

Each TC must be independently runnable and produce an unambiguous pass or fail.
Prefer `grep`, `dotnet build`, `hv_status`, or a direct DB query over manual inspection.

## Step 11 — Write `Retro/<SLUG>-FIX-NNN.md` (at runtime, on each test failure)

```markdown
# Fix <NNN> — <brief description>

## Failed test
Task <NNNN>, TC-<NNN>

## Root cause
<What was wrong.>

## Fix applied
<What was changed and why.>

## Outcome
Pass / still failing (→ escalate)
```

## Step 12 — Write `Results/<SLUG>-RESULT.md` and `Retro/<SLUG>-LESSONS.md` BEFORE hv_ship

`hv_ship` commits everything it finds — these files must exist before the ship call
or they land on the next branch and miss the PR.

### `Results/<SLUG>-RESULT.md`

```markdown
# Result: <workflow-name>

## Workflow ID
`<SLUG>`

## Outcome
SHIPPED | PARTIAL | FAILED

## Branch
`<generated-branch-name>`

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `<repo>` | TBD — filled from hv_ship output | — |

## Phase summary
### Phase 0 — Initialize
...
### Phase 1 — <name>
...
```

### `Retro/<SLUG>-LESSONS.md`

```markdown
# Lessons: <workflow-name>

## Workflow ID
`<SLUG>`

## What would have helped
<Numbered list. Be specific — name the missing MCP method, the doc that should
exist, the task instruction that was ambiguous, or the test that gave a false
positive. If nothing went wrong, write a single line saying so.>

## Fix files written
- FIX-001: <one-line summary>
- FIX-002: <one-line summary>
```

## Key rules

- **Results and Retro/LESSONS are written BEFORE `hv_ship`** — not after
- **FIX files are written before retrying** — never silently re-run a failed test
- **Task 0000 is always hv_status + hv_init/hv_next** — never skip it
- **Never use raw `git` for branching** — always hv_init, hv_next, hv_ship
- **See `toolkit/`** for reusable MCP call patterns (hive-deck, database, dotnet)
- **This scaffold was generated for deck `cloud-manager` on branch `amiable-pancake`**
- **Workflow folder:** `/Users/talker/workspace/cloud-manager/planning/execution-workflow`
- **Scaffold persisted at:** `/Users/talker/workspace/cloud-manager/planning/execution-workflow/cloud-manager/amiable-pancake/scaffold.md`
