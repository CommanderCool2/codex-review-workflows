# Codex Review Workflows

An installable Codex plugin containing two reusable review workflows:

- `iterating-plan-reviews` independently reviews an implementation plan against its approved design until the same reviewer reports no findings.
- **Iterative Code Review** (`iterating-code-reviews`) runs an independent subagent review-and-fix loop after implementation until the same reviewer returns `No findings.`

## Who this plugin is for

This plugin is for people using Codex to plan and implement changes in a local Git repository. The workflows depend on Codex skills, repository access, and agent/task tools; they are not portable prompt templates for arbitrary coding agents or IDE assistants.

| Workflow | Intended environment | Required capabilities |
|---|---|---|
| Plan review | Codex desktop app, CLI, or IDE extension | Local repository and file access, installed skills, and enabled subagents |
| Code review | Codex environment with a local workspace | Local Git and working-tree access plus enabled subagents |
| Other agents or IDEs | Unsupported unless adapted | Support for the plugin format and every tool contract used by the selected workflow |

Installing or copying the Markdown skill files is not enough if the environment cannot spawn and continue the required reviewer or expose the current repository state. In particular, do not install this plugin for the code-review workflow unless your Codex environment supports context-isolated subagents that share the local workspace.

## Prerequisites

Before installing, make sure that:

- Your work is in a local Git repository that Codex can read.
- Codex can read the repository instructions and run its relevant verification commands.
- Your environment provides the capabilities listed above for the workflow you want to use.
- You can allow for multiple review rounds. Each reviewer round uses additional model tokens and time.

Install the **Superpowers** plugin separately. Both iterative workflows use its `superpowers:receiving-code-review` and `superpowers:verification-before-completion` skills.

```powershell
codex plugin add superpowers@superpowers
```

## Install

Add this repository as a Codex plugin marketplace:

```powershell
codex plugin marketplace add CommanderCool2/codex-review-workflows
```

Then install the plugin:

```powershell
codex plugin add codex-review-workflows@codex-review-workflows
```

You can also open `/plugins` in the Codex CLI or the Plugins directory in the Codex desktop app, select **Codex Review Workflows**, and install the plugin there.

Start a new task after installation so Codex loads the bundled skills.

## Use

### Review an implementation plan

Use `iterating-plan-reviews` before implementation begins, after both the source specification and its implementation plan have been created or approved in the current task.

You need:

- An approved source specification stored in a file.
- An implementation plan stored in a separate file.
- Unambiguous paths to both files. Common locations are `docs/superpowers/specs/` and `docs/superpowers/plans/`, but those directories are not required.
- A Codex environment with subagents enabled.

The workflow treats the specification as fixed and the plan as editable. One context-isolated reviewer subagent reads the specification, plan, and repository directly. The main task checks every finding and updates the plan when the finding is valid; the same reviewer then rereads the revised file. You do not have to copy findings between agents. The workflow asks you only when a proposed correction would change an approved requirement, decision, or the specification itself.

Invoke the skill from the task in which the specification and plan were created or approved, and include their paths:

```text
Use $iterating-plan-reviews to review docs/superpowers/plans/FEATURE.md against docs/superpowers/specs/FEATURE.md until clean.
```

The plan is ready only after the same reviewer rereads the current files and returns the exact no-findings verdict required by the skill.

### Review a completed implementation with Iterative Code Review

Use `iterating-code-reviews` after completing the implementation of a documented plan and before final handoff.

You need:

- The completed implementation available locally on a feature branch.
- The approved source specification and implementation plan stored as files with unambiguous paths.
- A Codex workspace that exposes the current branch and working tree to enabled subagents.
- Relevant verification commands or evidence for the implementation.

The implementation does not have to be fully committed: the reviewer inspects both the current branch and its working tree, including uncommitted changes. The task invoking the skill first offers GPT-5.6 Sol with high reasoning, while allowing another supported model and reasoning effort. If you do not reply within 30 seconds, it selects GPT-5.6 Sol with high reasoning automatically. It then spawns one context-isolated reviewer subagent without a separate approval prompt and reuses that same subagent for every review round.

The implementation task remains the sole editor. The reviewer stays read-only. Valid findings are fixed and verified in the implementation task; incorrect findings receive evidence-based pushback. The workflow asks you before changing the approved specification, an approved decision, or the product scope.

Invoke the skill from the implementation task and include the artifact paths:

```text
Use $iterating-code-reviews to review this implementation using docs/superpowers/plans/FEATURE.md and docs/superpowers/specs/FEATURE.md.
```

The code review is complete only after the same reviewer rereads the current code and the complete relevant diff, returns `No findings.` with the required review evidence and no residual verification gaps, and the implementation passes final verification.

## Update

Refresh the marketplace checkout:

```powershell
codex plugin marketplace upgrade codex-review-workflows
```

Then update or reinstall the plugin from `/plugins` or the Codex desktop Plugins directory.

## Remove

Remove the installed plugin:

```powershell
codex plugin remove codex-review-workflows
```

Remove the marketplace if you no longer need it:

```powershell
codex plugin marketplace remove codex-review-workflows
```

## Repository layout

```text
.agents/plugins/marketplace.json
plugins/codex-review-workflows/
|-- .codex-plugin/plugin.json
`-- skills/
    |-- iterating-plan-reviews/
    |-- iterating-code-reviews/
    `-- verifying-implementation-plans/
```

## License

MIT
