# Repository guidance

## Skill source files

- This repository is the source of truth for the Codex Review Workflows plugin. Edit its checked-in skills under `plugins/codex-review-workflows/skills/` when changing a skill, and update repository documentation or metadata that describes the same behavior.
- Treat copies in a user's profile, such as `%USERPROFILE%\.codex\skills\`, and Codex plugin caches as installed runtime copies. Do not edit them for repository maintenance; plugin installation or update applies the repository version to future tasks once that version is available.
- Edit an installed copy only when the user explicitly asks to change the installed copy as well.
- Before editing a skill, locate its source with `rg` and confirm the target is inside this repository.
