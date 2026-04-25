# Long Spec Triage

Use this reference when the user provides a long PRD, multi-stage plan, large
GitHub issue, ticket tree, RFC, or detailed implementation brief.

The mandate stays the same: optimize for speed and a correct working result,
not maintainability. A long spec is input material, not an order to follow every
stage literally.

## Triage Goal

Convert the source document into the shortest safe execution plan:

- preserve explicit requirements and acceptance criteria
- preserve user-visible behavior
- preserve constraints that affect correctness, safety, or compatibility
- cut ceremony, future-proofing, generalized architecture, and nonessential
  sequencing
- expose independent tracks for parallel work
- verify outcomes, not process compliance

## Step 1: Extract Non-Negotiables

Read the spec once for structure, then extract:

- required user-visible behavior
- acceptance criteria
- data contracts or file formats
- compatibility constraints
- error cases and edge cases
- explicit "must not break" behavior
- security, privacy, data loss, or destructive-operation constraints

Everything else is negotiable unless the user says otherwise.

## Step 2: Classify The Spec

Label each section:

- **Must implement**: required for acceptance or correctness
- **Must verify**: test, smoke, or manual check required
- **Can collapse**: multiple stages can become one implementation task
- **Can cut**: nice-to-have, future-proofing, docs-only, cleanup, abstraction
- **Can parallelize**: independent file/module/subsystem ownership
- **Must sequence**: shared contract, shared file, migration, package metadata,
  generated artifact, integration glue

Do not ask the user to approve every cut. Ask only when cutting an item might
change observable behavior or violate an explicit acceptance criterion.

## Step 3: Build The Shortcut Plan

Write a short plan with:

- **Critical path**: the minimum ordered steps that make the feature work
- **Parallel tracks**: independent chunks that can go to separate agents
- **Corners cut**: exactly what will not be done and why it is safe for this
  disposable implementation
- **Bug matrix**: the checks needed to prove the requirements still hold

For a 20-stage PRD, the output might be 4-8 implementation steps plus a focused
verification matrix. Do not preserve the PRD's stage count unless the stages are
real runtime or deployment dependencies.

## Safe Corners To Cut

Usually safe for disposable open source or local patches:

- skip new abstraction layers
- copy nearby implementation patterns
- hardcode support for the specific version or narrow target environment
- implement the visible behavior directly at the integration point
- collapse design, implementation, and cleanup phases into one patch
- skip polished docs unless docs are part of the user-visible result
- skip generalized configuration when one fixed mode satisfies the request
- skip broad refactors and naming cleanup
- skip preserving compatibility with unshipped branch-only code

## Corners Not To Cut

Never cut:

- acceptance criteria
- edge-case behavior the user explicitly requested
- data-loss prevention
- authentication, authorization, or permission checks
- input validation that prevents crashes or corrupt output
- tests or smoke checks needed to prove the feature works
- baseline failure disclosure
- final integration verification after parallel work

## Planning Budget

Spend enough planning time to avoid slow rework:

- tiny request: a short checklist is enough
- medium feature: result contract plus 5-10 tasks
- long PRD or 20-stage plan: triage sections, identify cuts, then compress into
  a critical path and parallel tracks

Planning is successful only if it makes implementation faster. If the planning
artifact is becoming a second PRD, stop and compress it.

## Verification Against Long Specs

Verify against outcomes:

- make a checklist from the extracted non-negotiables
- map each requirement to a test, smoke step, or reviewer check
- run targeted checks for changed behavior
- run the broadest reasonable integration check after merging parallel work
- report any unverified requirement as a known risk, not as complete
