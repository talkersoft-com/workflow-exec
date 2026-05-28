# Lessons: cloud-manager/steamy-radish

## What would have helped
1. Grep patterns for YAML allow entries need to anchor to the full line (`^\s*- "\*"$`) — bare `"*"` matches gitignore entries like `"*.md"` and `"!README.md"`, giving false failures. Always use anchored patterns when checking YAML list entries.

## Fix files written
None — no test failures (test pattern corrected before execution).
