# Lessons: cloud-manager/enormous-kookaburra

## What would have helped

1. `teardown.Status` writes to `io.Writer` not stdout — knowing the internal package signatures before writing stubs would have saved one read-then-rewrite cycle on `status.go`.
2. `CONTRACT.md` being caught by the `*.md` gitignore rule was expected but required adding a negation entry — future operations contract docs should note this pattern.
3. `sortedStringKeys` and `walkListNode` existed in `main.go` as helpers that became unused after porting — the refactor naturally cleans them up, but the compiler errors were a useful signal to follow.
4. The stdin piped detection pattern (`os.Stdin.Stat()` + `ModeCharDevice` check) is idiomatic Go for detecting piped input — worth documenting in the ops contract for future callers.

## Fix files written

None.
