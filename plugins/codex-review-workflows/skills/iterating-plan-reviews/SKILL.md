---
name: iterating-plan-reviews
description: Use when an implementation plan and its source spec were created or approved in the current session and the user wants an independent AI review before implementation, especially when they do not want to relay findings or reread the artifacts.
---

# Iterating Plan Reviews

## Overview

Use one context-isolated reviewer as the plan's acceptance loop. The plan is not clean until that same reviewer rereads the current files and reports no findings.

**REQUIRED SUB-SKILL:** Use `verifying-implementation-plans` for the reviewer rubric.

**REQUIRED SUB-SKILL:** Use `superpowers:receiving-code-review` before revising the plan.

**REQUIRED SUB-SKILL:** Use `superpowers:verification-before-completion` before handoff.

## Workflow

1. Resolve the approved spec and plan from the current session using absolute paths. If multiple pairs are plausible, ask one focused question. Treat the spec as fixed and the plan as editable.
2. Spawn exactly one reviewer with `fork_turns="none"`. Provide only the repository and artifact paths, read-only scope, required rubric, and output contract below.
3. Read all findings. Verify each against the spec, repository, architecture, and approved decisions. Apply valid routine corrections to the plan; send technical pushback for invalid findings. Never apply feedback blindly.
4. If a finding is ambiguous, changes an approved decision, or requires editing the spec, request user authority. Continue unaffected work but keep completion blocked.
5. Send the revision to the same reviewer with `followup_task`. Summarize changes and require a fresh read from disk. Repeat while findings remain.
6. After the exact no-findings verdict, run document-integrity and repository-scope checks. Report the plan path, rounds, resolved findings, verdict, checks, and unrelated working-tree state.

## Reviewer Prompt

```text
Independently review SPEC_PATH against PLAN_PATH in REPOSITORY_PATH.
Do not edit files. Read the spec first, then the plan, and inspect repository
reality. Use verifying-implementation-plans. Return only anchored findings or
exactly: No findings. The plan appears consistent with the spec and technically
coherent from the reviewed architecture and implementation perspective.
```

## Decision Reference

| State | Action |
|---|---|
| Valid routine findings | Revise the plan; return it to the same reviewer |
| Invalid finding | Keep the plan; send technical reasoning |
| Spec or approved-decision conflict | Ask the user; block completion |
| Reviewer interrupted or unavailable | Report it; do not self-approve or silently replace the reviewer |
| Exact no-findings verdict | Run final checks and hand off |

## Red Flags

- "The deadline makes self-review sufficient."
- "First-round fixes do not need another review."
- "Use a new reviewer for fresher eyes."
- "The reviewer is slow; declare readiness."
- "Apply every finding to avoid debate."

Deadlines do not replace a clean verdict, reviewer continuity, or technical verification.

## Common Mistakes

- Passing conversation history instead of raw artifacts.
- Letting the reviewer edit files.
- Stopping after first-round fixes.
- Changing the approved spec without authority.
- Making the user relay routine findings.
- Claiming success after an interrupted review.
