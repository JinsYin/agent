---
name: gsx-add-todo
description: "Capture an idea, task, or issue surfaced mid-session as a structured todo — thin alias that hands off to /gsd:capture (default add-todo mode). Invoke when the user says 'add a todo', 'capture this as a todo', 'gsx-add-todo', 'remember to fix X later', 'log this for later work', or drops a task-shaped aside that isn't worth interrupting current work for."
argument-hint: "[optional: idea/task description — blank extracts from recent conversation]"
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Agent
  - AskUserQuestion
---

# gsx-add-todo

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Alias for `/gsd:capture` in its default mode (structured todo capture into `.planning/todos/pending/`; duplicate detection, area inference, STATE.md update, commit). Hand off immediately, no extra processing — do not prepend `--note`, `--backlog`, `--seed`, or `--list`, since those route to different workflows:

```
Skill("gsd:capture", args=$ARGUMENTS)
```
