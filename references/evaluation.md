# Evaluation Plan

Evaluate this skill on two axes:

1. Does it make agents move faster without skipping correctness evidence?
2. Does it trigger only when the user explicitly wants disposable speed over
   maintainable code?

## Behavior Evals

Use realistic repos or fixtures when possible.

### Case 1: Open Source App Feature

Prompt:

> Add keyboard shortcuts to this open source app. I do not care about code
> quality; I just need the shortcuts to work reliably and be tested.

Expected behavior:

- Skill creates a result contract for shortcut behavior and conflicts.
- It patches the shallowest input-handling layer.
- It adds targeted tests or browser smoke checks for supported shortcuts.
- It does not refactor the app's input architecture.
- Final report includes commands run and remaining risks.

### Case 2: Parallel Library Adapters

Prompt:

> Add support for three similar providers. They are independent. Go fast, use
> parallel agents, and make sure all edge cases work.

Expected behavior:

- Skill defines the shared provider contract sequentially.
- It dispatches one implementer per provider in separate worktrees.
- It runs targeted provider tests and final integration tests.
- It uses an integration reviewer before claiming completion.

### Case 3: Should Not Trigger

Prompt:

> Add billing support to our production app with clean architecture and a
> maintainable foundation for future payment methods.

Expected behavior:

- This skill should not be used.
- Route to normal architecture, implementation, and testing workflows instead.

### Case 4: Long PRD Compression

Prompt:

> Use quick-and-dirty on this 20-stage PRD. The requirements are detailed, but
> the code is throwaway. Find every safe corner to cut, parallelize what you can,
> and still make sure the acceptance criteria and edge cases pass.

Expected behavior:

- Skill reads the PRD enough to extract non-negotiable requirements and
  acceptance criteria.
- It does not execute all 20 stages literally.
- It classifies stages into must implement, must verify, can collapse, can cut,
  can parallelize, and must sequence.
- It produces a compressed critical-path plan with explicit shortcuts.
- It verifies against outcomes and reports any unverified requirement as a risk.

### Case 5: Trivial Patch (No Heavy Machinery)

Prompt:

> Quick-and-dirty: this open source CLI prints `version` instead of `--version`
> on `--help`. Just fix the one-line bug.

Expected behavior:

- Skill recognizes a trivial input and skips worktrees, subagents, and formal
  triage.
- It produces a one-line result contract.
- It edits the relevant file and runs a targeted check.
- It applies the shipping bar and stops as soon as the contract is satisfied.
- It does not invent extra edge cases or refactors.

### Case 6: Verification Loop Failure (Regression)

Prompt:

> Use quick-and-dirty to add CSV export to this view. Make sure all edge cases
> work — empty data, huge data, weird unicode, empty strings, nulls, very long
> strings, mixed types, malformed input, RTL, emojis, BOMs, escaping, line
> endings, locale, timezones, etc.

Expected behavior:

- Skill writes a result contract that names the user's edge cases.
- It implements the export.
- After two reviewer passes that surface no critical or likely-bug findings,
  it stops the edge-case loop.
- It records remaining concerns in a risk register and ships.
- It does not loop indefinitely on increasingly hypothetical scenarios.

### Case 7: Risky Cut Surfaces Confirmation

Prompt:

> Quick-and-dirty: take this PRD and ship the smallest version. Cut whatever
> isn't strictly required.

Where the PRD includes a "must require authentication" line.

Expected behavior:

- Skill triages the spec and identifies the auth line as a must-keep.
- If the smallest implementation would skip auth, the skill surfaces this as a
  single risky-cut confirmation question to the user.
- It does not silently cut authentication.

## Trigger Evals

Should trigger:

- "Hack this feature into this open source fork. I only care that it works."
- "Quick and dirty implementation please, but test all edge cases."
- "Just make this local patch work; we will never maintain it."
- "Use parallel agents and worktrees to get this disposable feature done fast."
- "I do not care about readability here, only zero known bugs."
- "Patch this library fork with the smallest working change."
- "Fast one-off implementation, no architecture ceremony."
- "This is throwaway code; maximize working result quality."
- "Here is a huge PRD; follow quick-and-dirty principles and cut every safe corner."
- "Build a quick prototype/POC/spike of this idea — throwaway code, just want to see if it works."
- "Monkey-patch this library to fix the bug locally; never going upstream."
- "Userscript to inject this behavior into the page; one-off."
- "Build me a personal browser extension hack to do X."
- "Internal-only tool, nobody else will touch it; ship it now."

Should not trigger:

- "Design a maintainable architecture for this feature."
- "Refactor this module for readability."
- "Create a public SDK API we can support long term."
- "Implement a database migration safely."
- "Harden production infrastructure."
- "Review this PR for maintainability."
- "Write a clean reusable component library."
- "Improve our coding standards."

## Assertions

For behavior evals, assert:

- A result contract is written before implementation.
- Worktree/subagent split is used only when ownership is independent and the
  orchestration overhead is justified.
- Trivial inputs do not trigger worktrees, subagents, or formal triage.
- Long PRDs are compressed into a critical path instead of followed stage by
  stage when stages are not runtime dependencies.
- Intentional cuts are listed and checked against acceptance criteria.
- Risky cuts are surfaced as a single confirmation question, not silently made.
- Verification commands are listed with outcomes.
- At least one edge-case review or checklist exists.
- The edge-case review loop stops when consecutive passes produce no critical
  or likely-bug findings.
- No completion claim appears without fresh verification evidence.
- The implementation avoids broad refactors unless required for correctness.
- A final risk register documents accepted risks instead of hiding them.

## Review Notes

Review transcripts, not only final output. The most important failure modes are:

- using "quick and dirty" as an excuse to skip tests
- overengineering despite the disposable constraint
- dispatching agents with overlapping file ownership
- trusting subagent success without independent verification
- hiding baseline failures
- imposing worktree/subagent ceremony on trivial requests
- looping on hypothetical edge cases without a shipping bar
- silently cutting requirements that needed user confirmation
