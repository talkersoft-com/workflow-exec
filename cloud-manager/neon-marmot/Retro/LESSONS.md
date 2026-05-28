# Lessons: cloud-manager/neon-marmot

## What would have helped
1. The `*bool` nil-means-true pattern was already in the codebase (`DeckConfig.EnableRootMCP`) — checking for prior art before designing the API saved a custom unmarshaller.
2. `ShouldOverwrite()` as a method on `ClaudeSettings` keeps the nil-check in one place; callers don't need to know the pointer semantics.

## Fix files written
None — no test failures.
