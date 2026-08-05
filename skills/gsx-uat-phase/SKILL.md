---
name: gsx-uat-phase
description: "Run conversational UAT on a built Phase — thin alias that hands off to /gsd:verify-work. Invoke when the user says 'UAT', 'verify', 'verify Phase', 'run UAT', 'gsx-uat-phase', 'accept this Phase', 'acceptance test'."
argument-hint: "[optional: phase number or acceptance scope description]"
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Agent
  - AskUserQuestion
---

# gsx-uat-phase

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Alias for `/gsd:verify-work` (conversational UAT: questionnaire, manual verify guide, gap recording, STATE.md updates). Hand off immediately, no extra processing:

```
Skill("gsd:verify-work", args=$ARGUMENTS)
```
