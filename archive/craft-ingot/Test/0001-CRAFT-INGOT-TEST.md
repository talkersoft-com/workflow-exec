# Test: Craft Ingot

> EXAMPLE FILE — This shows the pattern for a test file. Create a corresponding test file for every task you implement. Do not modify this file.

## Test ID
`0001-CRAFT-INGOT-TEST`

## Task Reference
`Tasks/0001-CRAFT-INGOT-TASK.md`

## When To Run
After completing all steps in the task file. Do not mark the task complete until this test passes.

## Test Cases

### TC-001: Ingot Present
- **Check**: Iron ingot exists in inventory
- **Pass**: Count >= 1
- **Fail**: Count == 0 → task did not complete; write an improvisation

### TC-002: No Ore Lost
- **Check**: Raw ore count matches pre-task baseline minus smelted quantity
- **Pass**: Every ore consumed produced a corresponding ingot
- **Fail**: Ore missing without ingot → something failed mid-cycle

### TC-003: Furnace Empty
- **Check**: Furnace input and fuel slots are empty after collection
- **Pass**: Both slots empty
- **Fail**: Residue remains → furnace may still be processing or jammed

## Scoring
All three test cases must pass. Partial pass is a failure. Do not proceed to the result file until all cases pass or you have documented the failure in an improvisation.

## On Failure
1. Write `Improvise/0001-CRAFT-INGOT-IMPROVISE-001.md` (or next available number)
2. Document exactly which test case failed and why
3. Adapt and retry
4. Re-run this test after the adaptation

## On Pass
Write `Results/0001-CRAFT-INGOT-RESULT.md` and mark the task complete.

---

> PATTERN NOTE — Test files are not optional. Every task has a test. The test defines what "done" means. If you cannot write a test for a task, the task is not specific enough — break it down further.
