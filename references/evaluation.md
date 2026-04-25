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
- Worktree/subagent split is used only when ownership is independent.
- Long PRDs are compressed into a critical path instead of followed stage by
  stage when stages are not runtime dependencies.
- Intentional cuts are listed and checked against acceptance criteria.
- Verification commands are listed with outcomes.
- At least one edge-case review or checklist exists.
- No completion claim appears without fresh verification evidence.
- The implementation avoids broad refactors unless required for correctness.

## Review Notes

Review transcripts, not only final output. The most important failure modes are:

- using "quick and dirty" as an excuse to skip tests
- overengineering despite the disposable constraint
- dispatching agents with overlapping file ownership
- trusting subagent success without independent verification
- hiding baseline failures
