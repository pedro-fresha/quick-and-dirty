# Shipping Bar

Use this reference when deciding whether the work is done. Quick-and-dirty
should be fast, not endless. "No known bugs" is a defined bar, not an
infinite loop on hypothetical edge cases.

## When To Stop

Ship when **all** of these hold:

1. Every item in the result contract has direct evidence of working: test
   output, smoke step, or an integration check.
2. Every explicitly named edge case has a test, smoke step, or reasoned
   reviewer note.
3. The bug matrix has been run end-to-end at least once with no critical or
   likely-bug findings outstanding.
4. The targeted automated tests touching the changed surface pass.
5. The broadest reasonable suite (changed-surface tests + fast smoke pass)
   passes, or any failures are demonstrably pre-existing baseline failures
   recorded before implementation began.
6. Any remaining concerns are documented as known risks, not as silent gaps.

If all six hold, stop. If only five hold, do the smallest thing that closes
the gap.

## Defining "No Known Bugs"

For disposable code, "no known bugs" means:

- The contract works on the inputs the user named.
- The contract works on the edge cases the user named.
- The contract works on a small set of inferred edge cases the agent or
  reviewer surfaced (null, empty, large, malformed, repeated).
- No code path silently swallows errors that would mask broken behavior.
- No test was disabled or weakened to make the suite pass.

It does **not** mean:

- All hypothetical edge cases regardless of likelihood.
- Bugs in code paths the user did not change.
- Pre-existing baseline failures in the upstream repo.
- Future-proofing against inputs the user did not request.

## Minimum Test Bar

Bound the test investment for disposable code.

Always write:

- one check per acceptance criterion that would silently fail without it
- one check per edge case the user explicitly named
- one smoke check that proves the user-visible result end-to-end

Usually skip:

- tests for behavior the platform, framework, or language already guarantees
- redundant unit tests when an integration test proves the same path
- test infrastructure refactors (no test base classes, helper extractions, or
  fixture frameworks unless the existing harness already requires them)
- snapshot tests for fluid output that the user did not explicitly require
- coverage targets — coverage is not the goal, contract evidence is

Prefer:

- the simplest harness already in the repo
- tests that read like the result contract, one assertion at a time
- integration tests over unit tests when both prove the same outcome

## Edge-Case Review Diminishing Returns

The edge-case hunter should produce findings the controller can act on, not a
list of hypothetical concerns.

Stop the edge-case loop when:

- The reviewer's last pass returned no critical or likely-bug findings.
- New findings are stylistic, speculative, or describe behavior the contract
  did not promise.
- Two consecutive passes produced the same set of findings.
- Findings concern code paths the user did not request and did not change.

Document anything left over as a known risk in the final report and ship.

## Risk Register Pattern

Use a risk register instead of grinding on every concern.

For each unresolved finding, record one line:

```
RISK: <one-line description>
- Severity: critical | likely | unlikely
- Trigger: <when this matters>
- Mitigation: <if any> | none
- Decision: fix-now | accept-for-disposable | escalate-to-user
```

Rules:

- `critical` cannot be `accept-for-disposable`. Either fix or escalate.
- `likely` should be fixed unless the user explicitly accepts it.
- `unlikely` is the right home for "I noticed but it's not in scope".
- Always show the register in the final report so the user sees what was
  accepted, not just what was fixed.

## Anti-Loop Rule

If the agent has run a verification command three times after small changes
without making progress, stop. Either:

- escalate to the user with the failing output and the hypotheses already tried
- swap the failing approach for a different one
- record the failure as a critical risk and let the user decide

Do not run the same agent prompt repeatedly without new evidence or a changed
hypothesis. The same applies to local fix loops.

## Final Report Shape

A complete report has:

- result contract with each item marked covered or risked
- corners intentionally cut, with one-line justifications
- verification commands run with outcomes
- known risks register
- single-line shipping decision: "ready" / "needs user decision" / "blocked"
