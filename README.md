# Codex Review Workflows

An installable Codex plugin containing two reusable review workflows:

- `iterating-plan-reviews` independently reviews an implementation plan against its approved design until the same reviewer reports no findings.
- `iterating-code-reviews` runs a visible, independent code-review-and-fix loop after implementation until the same reviewer returns `No findings.`

## Prerequisites

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

Invoke the plan workflow after a design specification and implementation plan have been approved:

```text
Use $iterating-plan-reviews to review the implementation plan against its approved spec until clean.
```

Invoke the code workflow after implementing a documented plan:

```text
Use $iterating-code-reviews to run an independent review-and-fix cycle on this implementation.
```

The code-review workflow asks which model and reasoning effort to use before it creates the visible reviewer task.

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
