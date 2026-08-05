---
name: gsx-add-backlog
description: "Park an idea that isn't ready for active planning into the ROADMAP.md backlog (999.x numbering) — thin alias that hands off to /gsd:capture --backlog. Invoke when the user says 'add this to the backlog', 'park this idea', 'gsx-add-backlog', 'not now but later', or wants to note something out-of-sequence without committing to a phase."
argument-hint: "[idea/feature description to park in the backlog]"
allowed-tools:
  - Read
  - Bash
---

# gsx-add-backlog

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.

Alias for `/gsd:capture --backlog` (adds a 999.x parking-lot entry to ROADMAP.md, creates the phase directory, commits). Hand off immediately, no extra processing:

```
Skill("gsd:capture", args="--backlog {description}")
```
