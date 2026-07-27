---
name: verifying-implementation-plans
description: Use when reviewing an implementation plan against a requirements, design, or feature spec to check exact coverage, consistency, sequencing, technical feasibility, architectural fit, or missing work before implementation starts.
---

# Verifying Implementation Plans

## Overview

Verify a plan against its source spec as a review task, not as a rewrite task. First reconstruct the requirements from the spec, then map the plan against them, then report only concrete findings the plan author must address.

The goal is to answer:

1. Does the plan fully and exactly cover the spec?
2. Does the plan contradict or dilute any requirement?
3. Is the plan technically sound, implementable, and sequenced correctly?
4. Are the findings phrased so they can be pasted directly back to the plan author?

## Inputs

Expect two primary artifacts:

- The implementation plan, usually under `docs/superpowers/plans/...`
- The source spec, usually under `docs/superpowers/specs/...`

If the prompt provides concrete file paths, use those exact files. Do not ask for alternatives unless a file is missing.

## Workflow

### 1. Read the spec first

Extract the spec into a concrete requirement inventory before judging the plan.

Capture:

- Functional requirements
- Non-functional requirements
- Explicit constraints
- Stated exclusions or non-goals
- UX or flow requirements
- Data/model/storage requirements
- Technical/architectural decisions already fixed by the spec
- Verification or rollout requirements

Do not collapse distinct requirements into one vague summary. Keep them atomic enough that you can test plan coverage requirement-by-requirement.

### 2. Read the plan second

Identify:

- Planned workstreams or phases
- Proposed architecture and data changes
- Dependencies and ordering
- Testing and verification steps
- Migration, rollout, or cleanup steps
- Assumptions the plan introduces beyond the spec

### 3. Build a coverage map

For each spec requirement, classify the plan as one of:

- `Covered`
- `Partially covered`
- `Missing`
- `Contradicted`
- `Over-scoped`

Use `Over-scoped` when the plan adds material work or architectural commitments that are not required by the spec and are not justified as enabling work.

### 4. Review technical quality

Even if coverage looks good, inspect whether the plan is technically defensible.

Check for:

- Missing prerequisite work
- Wrong sequencing or dependency order
- Hand-wavy implementation steps
- Architecture that conflicts with the current system or spec direction
- Data model changes without migration/backfill implications
- UI work without state/data/API support
- API or backend work without auth, validation, error handling, or access-control consideration
- Test plans that do not verify the risky parts
- Rollout steps missing for high-impact changes
- Risky assumptions presented as settled facts

### 5. Separate findings from rewrites

Do not silently rewrite the plan. Produce findings that explain:

- what is wrong
- why it is wrong
- what the plan needs to add, remove, or clarify

## Finding Types

Use these buckets:

- `Coverage gap`: required by spec, absent or incomplete in plan
- `Contradiction`: plan conflicts with explicit spec intent
- `Ambiguity`: plan wording is too vague to verify or execute safely
- `Technical risk`: proposed approach is likely unsound or incomplete
- `Architecture issue`: structure, ownership, boundaries, or dependencies do not hold up
- `Sequencing issue`: steps are in the wrong order or omit prerequisites
- `Verification gap`: tests or validation do not cover the real risk
- `Scope issue`: plan adds unnecessary work, or ignores explicit non-goals

## Output Format

Lead with findings. Keep them copy-paste ready for the plan author.

Use this shape:

```markdown
Findings

- High: [short title]
  Spec: [file + requirement or section]
  Plan: [file + relevant step or section]
  Issue: [precise mismatch or technical problem]
  Required update: [what the author should change in the plan]
```

If there are no issues, say:

```markdown
No findings. The plan appears consistent with the spec and technically coherent from the reviewed architecture and implementation perspective.
```

After findings, optionally add:

- `Open questions` only for true unknowns that block confident review
- `Coverage summary` as a short paragraph, not a giant matrix

## Review Standard

Be strict about exactness:

- If the spec says a behavior must exist, a vague nearby step is not enough.
- If the plan implies a different behavior, count it as a contradiction, not a partial match.
- If the plan skips operational or architectural consequences of its own changes, count that as a technical finding.
- If a requirement is covered only by unstated assumptions, count it as missing or ambiguous.

Prefer fewer high-signal findings over a long noisy list.

Always anchor each finding to concrete evidence from both artifacts. Prefer file path plus section heading or step title over vague references like "the spec says" or "the plan mentions".

## Common Failure Modes

- Treating the plan's wording as proof of coverage without checking the actual requirement
- Approving a plan that covers happy-path UI but not data, auth, or verification work
- Missing that the plan quietly changes scope or architecture beyond the spec
- Confusing "mentioned" with "fully planned"
- Accepting implementation steps that cannot be executed in the stated order

## Practical Heuristics

- Compare nouns and verbs: entities, states, actions, and transitions in the spec should all appear in the plan in executable form.
- Look for edge-case requirements in the spec; these are often the first things plans drop.
- Look for irreversible changes: schema changes, migrations, auth rules, and data derivations need explicit handling.
- Look for integration seams: UI, server actions, database, background jobs, and analytics often require coordinated changes across layers.
- If the plan proposes a pattern different from the spec's implied architecture, require the plan to justify it explicitly.
