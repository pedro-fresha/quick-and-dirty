# Worktree Parallelization

Use this reference when the task can be split across subagents or isolated git
worktrees.

## Fast Decision

Use worktrees when:

- coding on `main` or `master`
- current working tree has unrelated changes
- two or more implementers will edit code in parallel
- a subagent needs freedom to run tests, install dependencies, or iterate

Skip worktrees only for trivial, single-file work on the correct branch with a
clean working tree.

## Setup Sequence

1. Check the current repo state:

```bash
git status --short
git branch --show-current
```

2. Pick the worktree location:

- Use existing `.worktrees/` if present.
- Else use existing `worktrees/` if present.
- Else use `.worktrees/` and make sure it is gitignored.

3. Ensure project-local worktrees are ignored:

```bash
git check-ignore -q .worktrees || printf "\n.worktrees/\n" >> .gitignore
```

4. Create one branch and worktree per implementer:

```bash
git worktree add ".worktrees/<task-name>" -b "agent/<task-name>"
```

5. Install dependencies and run a baseline check in each worktree when feasible:

```bash
cd ".worktrees/<task-name>"
# Run the repo's normal setup and the fastest meaningful baseline check.
```

If baseline tests fail, record the failure before implementation. Do not let a
subagent claim it introduced or fixed unrelated baseline failures without proof.

## Decomposition Rules

Split by ownership. Each implementer must own a non-overlapping file or
directory set.

Good split dimensions:

- UI component, API route, CLI command, adapter, parser, test group
- separate package or module
- separate failing test file
- separate platform integration

Keep these sequential in the controller worktree:

- shared type/schema updates
- package manager metadata
- generated files
- migrations
- cross-cutting configuration
- final conflict resolution

## Controller Responsibilities

The controller agent owns coordination. Do not outsource these:

- final requirement interpretation
- task boundaries and file ownership
- creation and cleanup of worktrees
- merge order
- final full verification
- final user-facing status

## Merge Protocol

After each implementer reports back:

1. Inspect its diff and summary.
2. Run that track's targeted verification in its worktree.
3. Merge branches into the integration branch one at a time.
4. Resolve conflicts in favor of the result contract, not code neatness.
5. After all merges, run integration verification from the final workspace.
6. Remove worktrees only after verification:

```bash
git worktree remove ".worktrees/<task-name>"
git branch -d "agent/<task-name>"
```

## Speed Tactics

- Dispatch read-only research agents while implementers code.
- Dispatch edge-case hunters as soon as the result contract is stable.
- Prefer targeted tests during implementation and broad tests after integration.
- Timebox exploration. If the existing architecture is messy, patch the visible
  behavior instead of redesigning it.
- If two implementers need the same file, stop and make that shared edit
  sequentially.

## Failure Handling

When an implementer is blocked:

- missing context: provide files, logs, or exact commands and retry
- test failure: dispatch a debugging agent with the failing command and output
- scope collision: reassign ownership or merge sequentially
- ambiguous requirement: ask the user only if no safe assumption exists

Never run the same failing agent prompt repeatedly without new evidence or a
changed hypothesis.
