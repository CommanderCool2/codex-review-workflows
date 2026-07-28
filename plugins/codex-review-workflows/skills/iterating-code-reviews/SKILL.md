---
name: iterating-code-reviews
description: Use when an implementation task has completed work from a documented plan and design spec and needs to initiate an independent review before final handoff. Do not use when already assigned to perform that review.
---

# Iterating Code Reviews

## Overview

Use one visible, context-isolated Codex task as the read-only reviewer. Keep the implementation task as the sole editor and continue until that same reviewer rereads the current code and returns an evidence-bearing clean verdict.

**REQUIRED SUB-SKILL:** Use `superpowers:receiving-code-review` before acting on findings.

**REQUIRED SUB-SKILL:** Use `superpowers:verification-before-completion` before handoff.

## Non-Negotiable Contract

Follow these rules literally:

1. On invocation, ask only the exact question below and end the turn. Do not offer a menu, recommend alternatives, infer the answer from an earlier preference, or dispatch anything before the reply.

   > Use GPT-5.6 Sol with high reasoning for the review task? Reply `yes`, or specify another model and reasoning effort.

2. After the reply, create exactly one visible Codex task with `create_thread`. Never use `spawn_agent`, `fork_thread`, a replacement reviewer, or a new reviewer per round. Continue the one returned `threadId` with `send_message_to_thread`.
3. A clean response must start with the exact line `No findings.` and include every required Review evidence field with `Residual verification gaps: none`. A bare `No findings.` or an evidence section with missing verification is incomplete; send it back to the same reviewer instead of accepting it.
4. Never archive the reviewer task. Leave it visible after completion.

Violating the letter of this contract violates the workflow.

## Workflow

The workflow below begins only after the user answers the invocation gate. Interpret `yes` as model `gpt-5.6-sol` with reasoning `high`. Otherwise use the model and reasoning the user specifies. Normalize an obvious display name to the tool's model identifier; if the selection is ambiguous or unsupported, ask the user to correct it rather than silently substituting a model.

1. Resolve the implementation plan and source design spec from the current conversation. Convert them to absolute paths and verify they exist. If either is missing or multiple pairs are plausible, ask one focused question.
2. Read repository instructions and establish the exact review target:
   - Resolve and record the base ref/SHA and head SHA. If the intended base is ambiguous, ask one focused question instead of guessing.
   - Include the committed `BASE..HEAD` diff plus staged and unstaged working-tree changes.
   - Build a changed-file inventory using neutral facts from those diffs.
   - Put behavior and verification reported by the implementer under **Unverified implementer claims**. These are hypotheses for the reviewer to challenge, not review evidence.
3. Use the Codex task tools, not a subagent or fork:
   - Call `list_projects` and select the project containing the current repository.
   - Call `create_thread` with the selected model and reasoning, a project target, and the `local` environment so the reviewer sees the current checkout and uncommitted changes.
   - Keep the returned `threadId` and `hostId` for the entire loop.
4. Use the Reviewer Prompt Contract below as the initial `create_thread` prompt. Observe its result with `wait_threads`; use its cursor on later waits. Leave reviewer user-input or approval requests visible to the user, and answer ordinary reviewer questions with `send_message_to_thread` when the answer is already authoritative in the repository or approved artifacts.
5. Validate the response contract before treating the round as complete. Require findings or the exact clean first line, both review passes, every Review evidence field, and an explicit residual-gap statement. Ask the same reviewer to complete an incomplete response; do not infer the missing evidence.
6. Triage every finding against the spec, plan, repository conventions, code, and evidence. Apply valid in-scope findings in the implementation task. Send technical pushback for incorrect findings. Ask the user before changing an approved decision, editing the spec, or expanding product scope. Never let the reviewer edit tracked files and never apply feedback merely to obtain approval.
7. After fixes, run the narrowest relevant verification required by the repository. Send the same reviewer a concise disposition of every finding, the updated exact review target and changed-file inventory, and any unverified implementer claims. Require a fresh read of the current files and complete relevant diff, not merely the dispositions or fix summary.
8. Repeat with the same reviewer while findings or verification gaps remain. Do not impose a round limit, replace the reviewer, self-approve, or stop after fixing the first round.
9. Completion requires that same reviewer, after its latest reread, to return the complete evidence-bearing clean response with no residual verification gap. Then run the repository's final verification requirements. If final verification causes a code change, return the changed tree to the same reviewer again.
10. Report the reviewer task, selected model/reasoning, rounds, resolved or rejected findings, exact verdict, review evidence, and final verification. Leave the reviewer task visible and unarchived.

## Reviewer Prompt Contract

```text
You are the reviewer already dispatched by iterating-code-reviews. Do not invoke
that skill, create or fork another reviewer, delegate the review, or follow its
orchestration workflow. Perform the review directly.

Review REPOSITORY_PATH. Do not modify tracked files or implement fixes. You may
run focused verification that creates known generated artifacts and may safely
remove those generated artifacts when repository instructions permit it.

Implementation plan: PLAN_PATH
Design spec: SPEC_PATH

Exact review target:
- Base ref/SHA: BASE
- Head SHA: HEAD
- Committed diff: BASE..HEAD
- Staged changes: STAGED_STATE
- Working-tree changes: WORKTREE_STATE

Changed-file inventory:
NEUTRAL_DIFF_INVENTORY

Unverified implementer claims:
IMPLEMENTER_CLAIMS

Treat implementer claims as hypotheses, not independent review evidence. Read
the repository instructions, establish the stated diff baseline yourself, and
report any discrepancy. Complete both passes below before any verdict.

Pass A - requirements and regression coverage
1. Enumerate every new, removed, and explicitly preserved behavior in the plan
   and spec.
2. Map each behavior to its implementation and name the exact test assertion or
   code invariant offered as evidence. Apply a counterfactual: could a plausible
   regression in the changed surface violate the behavior while that evidence
   still passes? If yes, it is not proof. For a visual-text requirement, DOM
   text content and ancestor visibility do not prove the label itself is visibly
   rendered; require label-level geometry, computed-style, or visual evidence.
3. Report missing coverage and assertions that do not prove a behavior added,
   removed, or newly exposed to regression by this target. This includes
   unchanged state cleanup or concurrency safeguards that a new interaction
   path now relies on. When a native input becomes hidden behind a proxy action,
   reset and same-value reselection are newly exposed interaction behavior even
   if the reset statement itself is unchanged. Passing tests are insufficient
   when their assertions are not meaningful evidence.
4. For a property that the available test layer cannot faithfully observe, or
   that a well-covered shared component default directly guarantees, use code
   and neighboring-pattern inspection as the invariant. Do not invent brittle
   class assertions or pretend a synthetic event proves browser trust timing.
   Current render-tree inspection plus the remnant audit may prove removal of
   ordinary static presentation; do not require a separate negative assertion
   for every former label or string unless the plan/spec names that assertion
   or absence is an authorization, security, or accessibility boundary. A
   simple render branch plus same-group/order evidence may likewise prove static
   adjacency; do not demand an exact-sibling assertion solely for a hypothetical
   future insertion unless literal siblinghood controls behavior or the
   plan/spec names that exact assertion.
5. Record unchanged pre-existing coverage debt as context, not a finding,
   unless this target worsens it or the plan/spec names a new assertion for
   that unchanged branch. A preservation statement alone does not convert
   historical test debt into an in-scope finding. Do not combine historical
   debt with an actionable gap or demand duplicate test-layer coverage when
   existing tests exercise the relevant contract.

Pass B - adversarial quality review
1. Inspect the complete target diff and relevant surrounding code.
2. Evaluate likely bugs and failure modes; concurrency and state handling;
   error paths; accessibility; type and API contracts; security; performance;
   maintainability; unnecessary remnants; and repository conventions.
3. For each removed UI relationship or responsibility, trace its former IDs,
   props, state, imports, helpers, and documentation; report leftovers that now
   have no consumer or purpose.
4. For every added or changed test, record its timeout, retry, setup, cleanup,
   and assertion-pattern comparison with neighboring tests. A passing run does
   not clear a convention mismatch. For compilation-sensitive browser tests,
   flag a missing timeout allowance when equivalent neighboring tests use one,
   even if the current cold run happens to finish under the default timeout.
5. Include actionable Low findings that reduce regression risk or remove a
   concrete maintainability hazard. Exclude taste-only preferences.

Run the narrowest relevant independent verification for each changed test
category unless repository instructions prohibit it:
- changed unit/component tests: run the affected test files;
- changed Playwright/layout tests: run the focused browser test or grep;
- compilation-sensitive browser tests: compare neighboring timeout conventions
  and test a cold state when safe and relevant;
- changed server/client boundaries or routing: run the applicable typecheck or
  build check.

Do not repeat every full-suite gate merely because the implementer reports it.
Run focused checks capable of challenging the changed behavior. If required
focused verification cannot run, record the exact gap and do not present an
unqualified clean verdict.

For every actionable issue, provide an anchored file/line or smallest relevant
symbol, impact, Critical/High/Medium/Low severity, and concrete improvement.
Use an example implementation only when it materially clarifies the fix.

Use exactly one of these response shapes:
- Findings exist: put findings first, ordered by severity.
- No findings and no verification gaps: use `No findings.` as the exact first
  line.
- No findings but verification remains blocked: use `Review incomplete.` as the
  exact first line.

Then always provide:

Review evidence:
- Review target: VERIFIED_BASE..VERIFIED_HEAD plus staged/working-tree state
- Requirements checked: behavior -> implementation -> exact assertion/invariant -> counterfactual result
- Review passes completed: Pass A requirements/coverage; Pass B adversarial
- Changed-test convention audit: test -> neighboring timeout/retry/setup/cleanup/assertion comparison
- Removed-remnant audit: removed relationship/responsibility -> former support -> current consumer or finding
- Pre-existing coverage context: none | unchanged gaps excluded from findings
- Files and surrounding patterns inspected: ...
- Focused verification run: command -> result
- Residual verification gaps: none | exact gap and impact
```

## Follow-Up Contract

Send fixes and disputed-finding dispositions to the same task:

```text
Continue as the same directly reviewing reviewer. Do not delegate or implement
fixes. Reread the plan, spec, repository instructions, current files, and the
complete updated diff from disk. Do not review only the summary below.

Updated exact review target:
- Base ref/SHA: BASE
- Head SHA: HEAD
- Committed diff: BASE..HEAD
- Staged changes: STAGED_STATE
- Working-tree changes: WORKTREE_STATE

Updated changed-file inventory:
NEUTRAL_DIFF_INVENTORY

Resolved findings:
- FINDING: DISPOSITION

Disputed or blocked findings:
- FINDING: EVIDENCE OR USER DECISION

Unverified implementer claims:
IMPLEMENTER_CLAIMS

Repeat Pass A and Pass B from the original contract against the current target.
Rerun the focused verification relevant to changed tests and fixes. Return
remaining findings followed by every Review evidence field. If none remain and
there are no residual verification gaps, use `No findings.` as the exact first
line. If none remain but a verification gap persists, use `Review incomplete.`
as the exact first line.
```

## Decision Reference

| State | Action |
|---|---|
| Valid, in-scope finding | Fix in the implementation task, verify, and re-review |
| Incorrect or preference-only finding | Send evidence-based pushback and require reassessment |
| Spec change or scope expansion | Ask the user; keep completion blocked |
| Missing evidence field or bare `No findings.` | Ask the same reviewer to complete the response |
| Required focused verification unavailable | Keep completion blocked and require the exact gap and impact in Review evidence |
| Reviewer unavailable | Report the blocker; do not silently replace it |
| Evidence-bearing `No findings.` with no gaps | Run final verification and hand off |

## Red Flags

- Choosing the default model without asking, or mapping `yes` to anything other than GPT-5.6 Sol with high reasoning.
- Reviewer invokes `iterating-code-reviews`, delegates, or starts another reviewer.
- Review target omits the base/head, staged changes, or working-tree changes.
- Implementer claims are presented as independent evidence.
- Reviewer skips the requirements map, adversarial pass, assertion audit, or a relevant focused verification category.
- Concrete Low regression or maintainability hazards are discarded as preferences.
- A bare `No findings.` or a clean verdict with verification gaps is accepted.
- Letting the reviewer edit tracked files, starting a fresh reviewer after fixes, or archiving the reviewer task.
- Applying every suggestion to end the loop faster.

Any red flag means the review contract has not been satisfied.

## Rationalizations

| Shortcut | Reality |
|---|---|
| "The implementer already ran the full suite." | Implementer results are claims; the reviewer runs narrow checks that challenge changed behavior. |
| "The tests pass, so their assertions are adequate." | Pass A must show which assertion proves each behavior. |
| "Low issues are optional." | Concrete regression risks and maintainability hazards are actionable even at Low severity. |
| "No findings is self-explanatory." | A clean verdict is accepted only with the required review evidence and no gaps. |
| "A fresh reviewer is faster or more independent." | Reviewer continuity is part of acceptance; use the same task. |
| "Archive after completion to reduce clutter." | The visible task is the requested audit trail; leave it unarchived. |
