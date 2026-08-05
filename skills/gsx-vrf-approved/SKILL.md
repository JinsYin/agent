---
name: gsx-vrf-approved
description: "Release a GSD `checkpoint:human-verify` gate: resolve the Plan number (e.g. Phase 4.4) or Task number (e.g. Task 4.4.3; inferred from context / STATE.md if blank), validate it is an un-released checkpoint:human-verify gate, then reply the `<resume-signal>` it requires (usually 'approved') to unblock GSD. Locate→validate→reply only; no edits, no SUMMARY, no commit. Invoke when the user finished a checkpoint round and wants to release — 'approve', 'approved', 'passed, release it', 'this checkpoint is good', 'let GSD continue', 'gsx-vrf-approved'. Sibling: gsx-vrf-autorun runs the verify steps; gsx-vrf-review summarizes the round."
argument-hint: "[optional: GSD Plan number (e.g. Phase 4.4) or Task number (e.g. Task 4.4.3); inferred from context / STATE.md if blank]"
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
---

# gsx-vrf-approved

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Release a blocked GSD `checkpoint:human-verify` gate (`gate="blocking"`) after a round passed. Validate the target is a real, un-released checkpoint:human-verify gate, **then** reply the `<resume-signal>` (typically `approved`) — so GSD continues close-out (run `<verification>` → write `SUMMARY.md` → update `STATE.md` → commit docs). Validating first prevents signaling a Plan with no such gate or an `auto` task, which would corrupt the flow.

**Boundary:** only "locate → validate → reply resume signal". No code edits, file writes, verification scripts, SUMMARY, commits, or branch switches — GSD handles close-out after the signal.

> Prerequisite: the user should have **actually run** `<how-to-verify>` and confirmed pass. This skill does not test on their behalf. If issues were found, **do not use this skill** — describe the problem to GSD directly instead.

## 0 — Parse arguments, locate the target gate

Numbering: `X.Y` = Phase X / Plan Y; `X.Y.Z` = Phase X / Plan Y / Zth `<task>`.

**`$ARGUMENTS` has a number** — normalize (case / `Phase`·`Plan`·`Task` prefix / whitespace optional):

| User writes | Parsed as |
|-------------|-----------|
| `Phase 4.4`, `Plan 4.4`, `4.4`, `04-04` | Phase 04 / Plan 04 → locate Plan file |
| `Task 4.4.3`, `4.4.3` | Phase 04 / Plan 04 / 3rd task |
| `Task 3` | 3rd task in active Plan (from context / STATE.md) |

Locate via glob (zero-pad to 2 digits; Plan may have description/letter suffix, e.g. `04-03a-PLAN.md`):
```bash
ls .planning/phases/04-*/04-04*-PLAN.md
```
0 or multiple matches → list candidates, ask user, stop.

**`$ARGUMENTS` empty** — infer:
1. Conversation: did the executor just return a `checkpoint:human-verify` gate message? Its Plan is the target.
2. Else read `.planning/STATE.md` for the `executing` active phase; glob its `*-PLAN.md`; keep ones with a `checkpoint:human-verify` task and **no `*-SUMMARY.md`** in the same dir.
3. Exactly 1 → use it. 0 or multiple → "Could not uniquely locate a pending checkpoint:human-verify gate — please specify a Plan / Task number." and stop.

No `.planning/` → not a GSD project; tell user and stop.

## 1 — Validate it is an unreleased checkpoint:human-verify gate

Read all `<task>` of the located Plan (document order); check each `type`.

**Plan number (`X.Y`):**
- Has ≥1 `type="checkpoint:human-verify"` → pass; record index, `<name>`, `<what-built>`, `<resume-signal>`.
- No `checkpoint:*` task (pure `auto`) → refuse: "Plan 0X-0Y has no checkpoint:human-verify task and needs no human release — fully-automatic Plan, executor closes it out."

**Task number (`X.Y.Z`):**
- Zth is `checkpoint:human-verify` → pass.
- Zth is `auto`/other → refuse and **point to the real gate**: "Task X.Y.Z is an auto task ('<name>') — not a human-verify gate. The gate is Task X.Y.N ('<gate name>'); release that one."
- Zth doesn't exist (Z > task count) → refuse; tell user the Plan's task count.

**Already released:** a `*-SUMMARY.md` exists in the same dir → likely released already. **Do not** re-signal. Ask: "Plan 0X-0Y already has a SUMMARY.md — gate appears released/closed. Release again?" Wait for explicit confirmation.

`gate="blocking-human"` gates (e.g. npm legitimacy check): releasable — the user invoking this skill is the "human-verified" act — but in step 2 display `<how-to-verify>` items **in full** so the user can confirm they checked them.

## 2 — Display release scope, reply resume signal

Show a concise auditable summary:
```
## Releasing checkpoint:human-verify — Plan 0X-0Y / Task X.Y.Z
- Gate task:       Task N · <task name>
- What was built:  <1-2 sentence summary>
- Resume signal:   <resume-signal verbatim>
```

**Read the exact word in `<resume-signal>`** (usually quoted, e.g. `approved`; some use `verified`) — reply that word, not a default. Output it as the **last standalone line**:
```
✅ Released the checkpoint:human-verify gate for Plan 0X-0Y (Task N). GSD can continue close-out (verification → SUMMARY.md → STATE.md → commit docs).

approved
```
> Last line = the signal. If `<resume-signal>` requires a different word, replace it.

If the session has **no** blocked GSD flow waiting (restarted / context lost), the signal won't auto-continue. Add: "No blocked GSD execution flow detected — to resume, re-run `/gsd:execute-phase <phase>`; it will detect this checkpoint is released and continue close-out."

## Refusal cases

Do not reply the signal — explain honestly and stop — when:
- No `.planning/` (not a GSD project).
- Plan/Task number matches no file/task.
- Target Plan has no `checkpoint:human-verify` task (fully-automatic).
- Given Task is not `checkpoint:human-verify` (also point out the real gate index).
- Args blank and cannot uniquely locate a pending gate.
- `SUMMARY.md` already exists — confirm with user first.

## Guardrails

- Read and reply only. No Edit/Write, build/test scripts, SUMMARY, commit, or branch switch — close-out is GSD's.
- Prerequisite: user actually completed `<how-to-verify>`. Don't accept on their behalf; if issues found, they describe problems instead of using this skill.
- Validation fails → refuse, don't approximate (wrong signal continues the wrong Plan/task).
- One checkpoint per run.
- Reply the exact word `<resume-signal>` requires; don't assume `approved`.
- Reply in Chinese.
