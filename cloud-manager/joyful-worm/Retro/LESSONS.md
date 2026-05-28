# Lessons: cloud-manager/joyful-worm

## What would have helped

1. **Tracked `.md` files missing after branch switch** — same issue as previous runs; `git checkout HEAD -- .` required each time. This is a recurring problem in execution-workflow and should be fixed in `hv_next` or the provisioning logic.

2. **Branch name in deck.md is stale by the time the workflow runs** — the scaffold captures the branch at generation time (`joyful-worm`), but the deck advances before the workflow executes. Task 0000 should always update deck.md to the actual current branch as its first action.

## Fix files written
None.
