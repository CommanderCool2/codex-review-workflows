# README Workflow Guidance Design

## Goal

Help prospective users determine whether the plugin will work in their environment before they install it, then show compatible users how to prepare and invoke each workflow.

## Audience and compatibility

The README will identify the plugin as an OpenAI Codex/ChatGPT plugin rather than a portable workflow package for arbitrary coding agents. A compatibility table will distinguish the two bundled workflows:

- Plan review is intended for Codex environments with installed skills, local repository access, and enabled subagent support.
- Code review is intended for a local Codex desktop workspace that exposes the visible project and task controls required by its reviewer loop.
- Other agents and IDE assistants are unsupported unless they implement the plugin format and all tool contracts used by the selected workflow.

Compatibility guidance will appear before installation instructions.

## Workflow documentation

Each main skill will receive its own section containing:

1. When to use it.
2. Required artifacts and repository state.
3. A concise description of the main-agent/reviewer-agent loop.
4. An invocation example.
5. The successful completion condition.

The plan-review section will require an approved source spec and an implementation plan stored as files. It will explain that the plan remains editable while the spec is treated as fixed, and that one context-isolated subagent repeatedly reviews the current plan.

The code-review section will require completed implementation work on a feature branch plus the corresponding spec and plan files. It will explain that uncommitted changes may be reviewed, that the implementation task remains the sole editor, and that one visible reviewer task is reused until it returns the exact clean verdict.

## Shared prerequisites

The README will retain the Superpowers dependency and add requirements for:

- A local Git repository that Codex can read.
- Repository instructions and relevant verification commands that Codex can execute.
- The plugin and Superpowers plugin loaded in a newly started task.
- Sufficient model usage for repeated independent review rounds.

Workflow-specific requirements will remain next to their respective instructions so users do not mistake them for universal requirements.

## Scope

Only `README.md` will be changed during implementation. Skill behavior, plugin metadata, installation commands, and workflow contracts will remain unchanged.

## Verification

The completed README will be checked against both main `SKILL.md` files and the bundled plan-verification skill. The final diff will also be reviewed for unsupported compatibility claims, inconsistent terminology, missing prerequisites, and outdated invocation examples.
