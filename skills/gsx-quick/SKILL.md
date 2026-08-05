---
name: gsx-quick
description: "Front-door for one small problem that runs /gsd:quick --discuss — a lightweight discussion phase before planning (surfaces assumptions, clarifies gray areas, captures decisions) instead of quick's default fast path. Use whenever the user wants a single scoped problem fixed with GSD guarantees and wants it thought through rather than blind-fixed: 'gsx-quick', 'fix this properly', 'discuss this small task before doing it', 'talk it through then fix it', 'I'm not sure what this should touch — clarify then fix', 'discuss this quick task first'. Arg = problem description; ask if blank. vs gsx-fast: that runs /gsd:fast inline for trivial one-liners (no planning, no discussion); this is the weightier tier for a real problem worth planning. vs /gsd:plan-phase: that's for multi-subsystem features needing a roadmap; this is one scoped problem."
argument-hint: "[problem description]"
allowed-tools:
  - Skill
  - AskUserQuestion
---

# gsx-quick

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.

---

A thin front-door: take the problem description and run `/gsd:quick --discuss`.

**Why it exists:** `/gsd:quick` defaults to the fast path — it skips discussion, which suits a task you already know cold. This wrapper exists for the opposite case, and its whole job is to make `--discuss` the default so a fuzzy problem gets discussed — assumptions surfaced, gray areas clarified, decisions captured in CONTEXT.md — before planning, rather than blind-fixed. That discussion phase lives entirely inside `/gsd:quick`; there is nothing for this wrapper to add on top.

**Boundary:** Orchestrate only. `/gsd:quick` owns everything — investigation, planning, the edits, the commits, `.planning/quick/{id}/`, STATE.md. Don't edit source, don't run git operations, don't re-plan or second-guess its questions here. If you feel the urge to "just fix it quickly," that's the signal to hand off, not to act.

## 0 — Get the problem description

`$ARGUMENTS` = the problem to fix. If empty, ask: *"这次要做什么？先用一句话描述这个问题。"*

Two cases are worth redirecting rather than force-fitting, because the tier matters more than the entry point:

- **Genuinely trivial** (typo, one constant, spacing/color, a single config line) → `gsx-fast` fits better; `--discuss` would spend a discussion round on a one-liner. Say so and stop unless the user still wants it here.
- **Large multi-subsystem feature** needing a roadmap → `/gsd:plan-phase` + `/gsd:execute-phase` fits better. Say so and let the user confirm before continuing.

Anything else — a real, scoped problem — goes to step 1.

## 1 — Call /gsd:quick --discuss

```
Skill("gsd:quick", args="--discuss " + problem description)
```

`--discuss` leads the args so `/gsd:quick` parses it as a flag rather than as part of the description.

`/gsd:quick` now owns the run: it discusses gray areas with the user, then plans, executes, and commits. Let it drive — including its questions, which are the point of `--discuss`. Never answer them on the user's behalf.

## 2 — Relay the outcome

`/gsd:quick` produces its own close-out (quick_id, commits, task_dir). Don't re-summarize the code work it already reported — add one line of framing and stop:

```
## gsx-quick — Done
问题：{problem}
交付：见上方 /gsd:quick 的 close-out（quick_id / commit / task_dir）。
```

If the user cancelled inside `/gsd:quick`, or it produced no commit, say so plainly — nothing landed, and this wrapper has nothing to clean up since it never touched the tree.

---

## Guardrails

- One problem per run. A different problem → restart.
- Reply in Chinese.
