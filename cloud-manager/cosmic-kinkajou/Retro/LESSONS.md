# Lessons: cloud-manager/cosmic-kinkajou

## What would have helped
1. When a binary is rebuilt and installed mid-session, the next `hv_init` call picks up the new code. If a behaviour change isn't observed after installing, check whether `hv_init` has been called with the updated binary — not whether the code is correct.
2. `filepath.Dir(plan.DeckDir)` as the workspace root write target is already in place. Future workspace-root writes don't need a code change — they need the config + binary to be in sync.

## Fix files written
None.
