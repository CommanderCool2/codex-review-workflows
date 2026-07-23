---
name: iterating-code-reviews
description: Use when implementation from a documented plan and design spec is complete and needs independent review before final handoff.
---

# Iterating Code Reviews

## Overview

Use one visible, context-isolated Codex task as the read-only reviewer. Keep the implementation task as the sole editor and continue until that same reviewer rereads the current code and returns the exact clean verdict.

**REQUIRED SUB-SKILL:** Use `superpowers:receiving-code-review` before acting on findings.

**REQUIRED SUB-SKILL:** Use `superpowers:verification-before-completion` before handoff.

## Non-Negotiable Contract

Follow these rules literally:

1. On invocation, ask only the exact question below and end the turn. Do not offer a menu, recommend alternatives, infer the answer from an earlier preference, or dispatch anything before the reply.

   > Use GPT-5.6 Sol with medium reasoning for the review task? Reply `yes`, or specify another model and reasoning effort.

2. After the reply, create exactly one visible Codex task with `create_thread`. Never use `spawn_agent`, `fork_thread`, a replacement reviewer, or a new reviewer per round. Continue the one returned `threadId` with `send_message_to_thread`.
3. Only the literal reviewer response `No findings.` is clean. `APPROVED`, `looks good`, `no actionable findings`, silence, and equivalent wording are not clean; ask the same reviewer to reread and return the required verdict.
4. Never archive the reviewer task. Leave it visible after completion.

Violating the letter of this contract violates the workflow.

## Workflow

1. Resolve the implementation plan and source design spec from the current conversation. Convert them to absolute paths and verify they exist. If either is missing or multiple pairs are plausible, ask one focused question.
2. Read repository instructions and inspect the current branch, working tree, implementation diff, and verification evidence. Produce a concise implementation summary grounded in the actual changes.
3. Apply the invocation gate above. `yes` means model `gpt-5.6-sol` and reasoning `medium`. Otherwise use the model and reasoning the user specifies. Normalize an obvious display name to the tool's model identifier; if the selection is ambiguous or unsupported, ask the user to correct it rather than silently substituting a model.
4. Use the Codex task tools, not a subagent or fork:
   - Call `list_projects` and select the project containing the current repository.
   - Call `create_thread` with the selected model and reasoning, a project target, and the `local` environment so the reviewer sees the current checkout and uncommitted changes.
   - Keep the returned `threadId` and `hostId` for the entire loop.
5. Use the reviewer prompt contract below as the initial `create_thread` prompt. Observe its result with `wait_threads`; use its cursor on later waits. The implementation task receives and processes the result directly. Leave reviewer user-input or approval requests visible to the user, and answer ordinary reviewer questions with `send_message_to_thread` when the answer is already authoritative in the repository or approved artifacts.
6. Triage every finding against the spec, plan, repository conventions, code, and evidence. Apply valid in-scope findings in the implementation task. Send technical pushback for incorrect findings. Ask the user before changing an approved decision, editing the spec, or expanding product scope. Never let the reviewer edit files and never apply feedback merely to obtain approval.
7. After fixes, run the narrowest relevant verification required by the repository. Send the same reviewer a concise disposition of every finding plus verification evidence. Require a fresh read of the current files and complete relevant diff, not merely the fix summary.
8. Repeat with the same reviewer while findings remain. Do not impose a round limit, replace the reviewer, self-approve, or stop after fixing the first round.
9. Completion requires the same reviewer to return exactly `No findings.` after its latest reread. Then run the repository's final verification requirements. If final verification causes a code change, return the changed tree to the same reviewer again.
10. Report the reviewer task, selected model/reasoning, rounds, resolved or rejected findings, exact verdict, and final verification. Leave the reviewer task visible and unarchived.

## Reviewer Prompt Contract

```text
Review the code changes currently visible in REPOSITORY_PATH on the current
branch and working tree. Do not edit files.

Implementation plan: PLAN_PATH
Design spec: SPEC_PATH

Implementation summary:
IMPLEMENTATION_SUMMARY

Read the repository instructions, then inspect the actual relevant diff,
changed files, surrounding architecture, and tests. Use the plan and spec as
requirements, but independently verify the summary against the code.

Focus primarily on long-term maintainability, stability, robustness, and code
quality. Evaluate architecture and design decisions, separation of concerns,
modularity, readability, consistency, naming, unnecessary complexity, error
handling, edge cases, reliability, likely bugs, test gaps and test quality,
type safety, API contracts, relevant performance and security concerns,
technical debt, and violations of repository conventions.

For every actionable issue:
1. Anchor it to a file and line or the smallest relevant symbol.
2. Explain why it is a problem.
3. Assign severity: Critical, High, Medium, or Low.
4. Propose a concrete improvement.
5. Provide an example implementation when it materially clarifies the fix.

Distinguish confirmed defects from questions or optional preferences. Return
findings first, ordered by severity. If there are no actionable findings,
return exactly: No findings.
```

## Follow-Up Contract

Send fixes and disputed-finding dispositions to the same task:

```text
Re-review the current code from disk and the complete relevant diff.

Resolved findings:
- FINDING: DISPOSITION AND VERIFICATION

Disputed or blocked findings:
- FINDING: EVIDENCE OR USER DECISION

Do not edit files. Report remaining actionable findings using the original
contract. If none remain, return exactly: No findings.
```

## Decision Reference

| State | Action |
|---|---|
| Valid, in-scope finding | Fix in the implementation task, verify, and re-review |
| Incorrect or preference-only finding | Send evidence-based pushback and require reassessment |
| Spec change or scope expansion | Ask the user; keep completion blocked |
| Reviewer unavailable | Report the blocker; do not silently replace it |
| Exact `No findings.` verdict | Run final verification and hand off |

## Red Flags

- Choosing the default model without asking.
- Using a hidden subagent or fork instead of a visible task.
- Letting the reviewer edit the shared checkout.
- Starting a fresh reviewer after fixes.
- Treating "looks good," approval, or silence as `No findings.`
- Archiving the reviewer task.
- Applying every suggestion to end the loop faster.
- Replacing the literal contract with an equivalent-sounding workflow.

Any red flag means the review contract has not been satisfied.

## Rationalizations

| Shortcut | Reality |
|---|---|
| "The user's usual model is known." | This invocation still requires the exact model question. |
| "A fresh reviewer is faster or more independent." | Reviewer continuity is part of acceptance; use the same task. |
| "APPROVED means the same thing." | Only the literal `No findings.` verdict satisfies the contract. |
| "Archive after completion to reduce clutter." | The visible task is the requested audit trail; leave it unarchived. |
