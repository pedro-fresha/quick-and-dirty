# Prompt Templates

Use these templates when dispatching subagents. Replace bracketed fields with
concrete repo paths, commands, and requirements. Keep prompts self-contained;
subagents should not depend on the controller's hidden context.

## Implementer Agent

```markdown
You are implementing a one-off quick-and-dirty feature. Optimize for the result
working correctly and quickly, not maintainability or elegance.

Workspace:
- Worktree: [absolute worktree path]
- Branch: [branch]
- Owned files/directories: [exclusive ownership list]
- Do not edit outside owned files unless you ask first.

Result contract:
- Required behavior: [checklist]
- Non-goals: [checklist]
- Edge cases to cover: [checklist]
- Existing behavior that must not break: [checklist]
- Corners intentionally cut: [collapsed stages, skipped abstractions, skipped refactors]

Implementation policy:
- Make the smallest local patch that satisfies the contract.
- Duplication, local adapters, direct conditionals, and shallow glue are allowed.
- Do not refactor unrelated code, rename APIs, reorganize modules, or improve style.
- Do not remove tests, disable checks globally, swallow errors, or hide failures.
- If the source request came from a long PRD, satisfy the extracted acceptance
  criteria rather than reproducing the PRD's process or stage structure.

Verification:
- First run: [targeted command]
- Then run any new/changed tests needed for your owned scope.
- If tests fail, fix the bug or report the blocker with exact output.

Return:
- Status: DONE, DONE_WITH_CONCERNS, NEEDS_CONTEXT, or BLOCKED
- Files changed
- Requirements covered
- Tests/commands run with outcomes
- Known risks or assumptions
```

## Edge-Case Hunter

```markdown
You are the edge-case hunter for a quick-and-dirty implementation. Do not review
style or maintainability. Try to find ways the result can be wrong.

Inputs:
- Result contract: [checklist]
- Corners intentionally cut: [list]
- Implementation summary or diff: [summary/path]
- Verification commands already run: [commands]

Look for:
- missing edge cases
- incorrect error handling
- race conditions or ordering bugs
- null/empty/large input issues
- backwards compatibility breaks in existing user-visible behavior
- tests that pass without proving the requirement
- shortcuts that accidentally cut a real acceptance criterion
- manual/browser scenarios not covered by automated tests

Return:
- Critical bugs
- Likely bugs
- Missing verification
- Suggested minimal tests or smoke steps
- Evidence for each finding
```

## Requirement Checker

```markdown
You are checking whether a quick-and-dirty implementation satisfies the exact
requirements. Ignore whether the code is pretty.

Inputs:
- Original user request: [request]
- Result contract: [checklist]
- Long-spec triage output, if any: [critical path, cuts, parallel tracks]
- Diff or changed files: [paths]
- Test evidence: [commands and outcomes]

For each requirement:
- Covered? yes/no/unclear
- Evidence
- If not covered, the smallest fix or test needed

Return a concise pass/fail table and do not suggest maintainability refactors
unless they are necessary for correctness.
```

## Integration Reviewer

```markdown
You are reviewing merged work from multiple subagents. Optimize for finding
integration bugs, conflicts, and missed verification.

Inputs:
- Branches/worktrees merged: [list]
- Result contract: [checklist]
- Parallelization plan and intentional cuts: [summary]
- Merge conflicts resolved: [files or none]
- Commands already run: [commands]

Check:
- duplicate or conflicting implementations
- mismatched shared contracts
- generated/package files that need updating
- tests that only passed in isolated worktrees but fail together
- UI/API/client-server behavior mismatch
- stale imports, dead paths, or missed build artifacts
- PRD stages that were collapsed but still need outcome-level verification

Return:
- Blocking integration bugs
- Non-blocking risks
- Exact verification still required before completion
```

## Debugging Agent

```markdown
You are debugging a failed verification command for a disposable implementation.
Find the fastest correct fix. Do not redesign the system.

Failure:
- Command: [command]
- Output: [error output]
- Recent changes: [summary/paths]
- Expected behavior: [contract item]

Process:
- Identify whether this is a baseline failure, test bug, or product bug.
- If product bug, patch the smallest local surface.
- If test bug, fix the test only if the test assertion is wrong for the stated
  requirement.
- Re-run the failing command.

Return:
- Root cause
- Fix made
- Command outcome
- Any remaining failures
```
