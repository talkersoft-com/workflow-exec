# Lessons: cloud-manager/pulsating-tulip

## What would have helped
1. The initial workflow scope ("comment out a line") was too narrow — the user's intent was schema alignment. Clarifying the architectural goal up front would have avoided the mid-task pivot.
2. Test scope should match feature scope: a config value change needs 1-2 test cases, not 5 task files. Simple = simple tests.
3. When a user says "match Claude's representation 1:1", that means struct tags need both `yaml:` and `json:` so the same type serialises correctly in both directions without a separate translation step.

## Fix files written
None.
