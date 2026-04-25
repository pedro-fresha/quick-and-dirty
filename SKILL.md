---
name: quick-and-dirty
description: >-
  Use when the user explicitly wants a one-off, disposable, quick-and-dirty
  implementation for an open source library, app, fork, local patch, prototype,
  POC, spike, hack, monkey-patch, userscript, browser-extension hack, internal
  tool, or throwaway feature where result correctness and speed matter far more
  than maintainability, readability, elegance, or long-term architecture. Also
  use for "hack it in", "just make it work", "never touched again", "fast
  implementation", "throwaway script", "ship it now", "smallest patch", "find
  every safe corner to cut", or "zero known bugs" requests, including when the
  input is a long detailed PRD that should be compressed into the fastest valid
  execution path. Do not use for durable product code, production infrastructure,
  security-sensitive changes, migrations, public API contracts, or work where
  future maintenance quality matters.
compatibility: Portable Agent Skills core; richer behavior when the host supports subagents and git worktrees.
---

# Quick And Dirty Implementation

## Purpose

This skill optimizes for fast, working results in disposable code. The goal is
not beautiful implementation. The goal is the smallest, fastest patch that
satisfies the requirements and has no known bugs after serious verification.

"Quick and dirty" means:

- Accept duplication, local glue, direct imperative code, and shallow patches.
- Prefer the smallest change surface that makes the feature work.
- Use parallel agents and worktrees when isolation pays off, and skip them when
  orchestration overhead would beat the savings.
- Spend saved design time on tests, edge cases, and evidence — bounded by the
  shipping bar, not by an infinite "what if" loop.

It does **not** mean:

- skipping tests
- hand-waving edge cases
- changing unrelated behavior
- hiding errors
- breaking existing workflows
- making destructive or security-risky changes

If the user's real priority is maintainable production code, route elsewhere.

## Adaptive Sizing

Match ceremony to the size of the request. Same mandates, scaled effort.

| Input shape | Planning effort | Subagents | Verification |
| --- | --- | --- | --- |
| Trivial (1 file, < 10 lines, clear intent) | One-line contract | None | Targeted test or smoke check |
| Small feature or fix | 3-5 line contract, ad-hoc checklist | Optional, only if useful | Targeted tests + bug matrix |
| Medium feature (multi-file, multi-step) | Result contract + 5-10 task plan | Use when ownership is independent | Targeted tests + integration smoke |
| Long PRD / multi-stage plan / large issue | Triage via `references/long-spec-triage.md` | Parallel tracks where safe | Outcome-mapped bug matrix + integration |
| Vague chat-style prompt | Restate the request as a contract first | Decide after sizing | Verify the restated contract |

Do not impose worktrees, multi-agent orchestration, or formal triage on tiny
requests. The mandates (no false success, no skipped edge cases, no destructive
changes) still hold; the machinery shrinks.

## Repo-Type Awareness

The disposable strategy depends on where the patch lands:

- **Open source fork**: changes will likely be rebased onto upstream. Keep
  changes localized, avoid touching files that change often upstream, prefer
  monkey-patches, plugin hooks, or wrapper modules over edits to core files.
- **Personal prototype / spike**: anything goes; cut the most corners.
- **Vendored dependency / `node_modules`-style copy**: changes will be wiped on
  reinstall. Either avoid editing the vendored copy, or document a postinstall
  reapply step.
- **Internal tool / one-off script**: maximum freedom; still verify outcomes.
- **User's own repo with later cleanup planned**: leave a short `TODO:
  quick-and-dirty` marker so future maintainers can find the disposable seams.

## Default Workflow

1. **Lock the result contract.**
   - Write a short checklist of required behavior, non-goals, and edge cases.
   - Ask only if an ambiguity blocks correctness. Otherwise make the fastest
     reasonable assumption and record it.
   - Prefer user-visible outcomes over architectural ideals.
   - If the input is a long PRD, plan, issue list, or staged spec, read
     `references/long-spec-triage.md` before decomposing. Do not execute the
     document stage-by-stage by default; compress it into the fastest route that
     still satisfies the mandates and acceptance criteria.

2. **Create an isolated execution surface.**
   - Use a feature branch or worktree before editing, especially in a dirty repo.
   - For parallel implementation, use one worktree per implementer.
   - Run enough baseline verification to distinguish pre-existing failures from
     new failures.
   - Read `references/worktree-parallelization.md` when using worktrees or
     subagents.

3. **Decompose for speed.**
   - Split by independent behavior, subsystem, file ownership, or test group.
   - Keep shared contracts, shared types, and integration glue sequential.
   - Dispatch implementers only when their file ownership will not overlap.
   - Explicitly list the corners you are cutting: skipped abstractions, skipped
     refactors, collapsed phases, reused patterns, manual setup, or narrow
     compatibility choices.
   - Use the prompt templates in `references/prompt-templates.md` for
     implementers, edge-case hunters, and integration reviewers.

4. **Implement the shallowest working patch.**
   - Reuse existing local style just enough to avoid accidental breakage.
   - Prefer localized conditionals, copied patterns, adapters, or small helper
     functions over new abstractions.
   - Avoid broad refactors, cleanup, renaming, framework swaps, or "while here"
     improvements.
   - Do not preserve compatibility with unshipped in-progress branch code unless
     the user asked for it.

5. **Test for bugs, not beauty.**
   - Build a bug matrix from the result contract: happy path, error path, edge
     cases, existing behavior, integration points, and user-visible smoke path.
   - Add automated tests when the project has a usable test harness.
   - Use targeted tests during implementation, then run the broadest reasonable
     suite before claiming completion. "Broadest reasonable" means the tests
     touching the changed surface plus a fast smoke pass — not the entire repo
     suite when that takes minutes for a tiny patch.
   - Apply the minimum test bar from `references/shipping-bar.md`: one check per
     acceptance criterion, one per explicitly named edge case, skip tests for
     behavior the platform already guarantees.
   - For UI work, include a browser or manual smoke path when possible.

6. **Run adversarial verification.**
   - Have at least one reviewer check requirements and edge cases, not code
     elegance.
   - If the task was parallelized, run an integration reviewer after merging
     worktrees.
   - Treat every reviewer finding as a bug until disproven by evidence.

7. **Report with evidence and ship.**
   - State what changed, which requirements are covered, which verification ran,
     and any remaining known risks.
   - Never claim "done", "fixed", "passing", or "no bugs" without fresh
     verification evidence.
   - Apply the shipping bar from `references/shipping-bar.md` to decide when the
     work is finished. "No known bugs" is a defined bar, not an infinite loop.

## Parallelization Rules

Parallelize by default when there are two or more independent workstreams.

Good parallel tracks:

- frontend UI and backend endpoint in separate files, after the request/response
  contract is defined
- multiple unrelated test failures
- independent provider adapters
- separate CLI commands or feature flags
- implementation and edge-case research in separate read-only agents

Do not parallelize:

- shared schema/type changes
- one file edited by multiple implementers
- unclear requirements
- migrations or destructive operations
- security-sensitive changes
- stateful integration work that must be reasoned about as one system
- tasks where the orchestration overhead (worktree setup, dispatch, merge,
  integration verification) is larger than doing the work sequentially —
  typically anything < ~15 minutes of sequential work, or fewer than 2 truly
  independent tracks

## Dirty-Code Budget

Allowed when it improves speed without increasing bug risk:

- copy-paste from nearby code
- one-off adapters
- small local helpers
- explicit branching instead of generalized strategies
- direct data shaping at the call site
- temporary compatibility for the exact target version
- adding focused tests without refactoring the system under test
- converting a long PRD into a shorter critical-path execution plan
- collapsing multi-stage rollout/design/documentation phases into direct
  implementation when they are not required for correctness
- implementing only the requested visible behavior instead of the PRD's implied
  future extensibility

Forbidden even for disposable work:

- swallowing exceptions or ignoring failed promises
- removing tests to pass the suite
- broad try/catch blocks that hide real failures
- hardcoded secrets or credentials
- changing public behavior outside the requested feature
- disabling lint/type/test gates globally
- racing shared state between parallel agents
- claiming success from an agent report without independent verification

## Stop And Escalate

Stop and ask the user before continuing when:

- the repo baseline is broken and blocks verification
- the requested feature requires a destructive data change
- correctness depends on credentials, production data, or secrets you do not have
- the only available implementation would be security-sensitive or irreversible
- agents report conflicting changes that cannot be merged safely

## Reference Router

- Read `references/shipping-bar.md` when deciding whether the work is done,
  when defining the minimum test bar, or when an edge-case review is producing
  diminishing returns.
- Read `references/worktree-parallelization.md` when dispatching subagents,
  creating worktrees, merging worktree branches, or deciding what can run in
  parallel.
- Read `references/long-spec-triage.md` when the user provides a PRD, long
  GitHub issue, multi-stage plan, ticket tree, or detailed implementation brief.
- Read `references/prompt-templates.md` when creating prompts for implementer,
  edge-case, spec-check, or integration-review agents.
- Read `references/evaluation.md` when evaluating this skill's behavior or
  trigger quality.
