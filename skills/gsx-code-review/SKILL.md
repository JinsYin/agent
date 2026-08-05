---
name: gsx-code-review
description: "Review + auto-fix on a GSD Phase's changed source — equals /gsd:code-review {N} --fix --all. Arg is the Phase number (inferred from context / STATE.md last completed phase if blank). Invoke when the user says 'code review Phase X', 'review and auto-fix Phase X', 'run code-review --fix', 'gsx-code-review'. Reviews the changed source itself (bug/security/quality) — unlike gsx-uat-planfix / gsx-uat-quickfix which fix acceptance Gaps in VERIFICATION.md / UAT.md."
argument-hint: "[optional: Phase number, e.g. 4 / 04 / Phase 4; inferred from context or STATE.md if blank] [optional passthrough: --depth=quick|standard|deep / --files=a,b]"
allowed-tools:
  - Read
  - Glob
  - Bash
  - Skill
  - AskUserQuestion
---

# gsx-code-review

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Thin orchestrator: parse Phase number → call `/gsd:code-review {NN} --fix --all` → relay result. The command does all reviewing/fixing/REVIEW.md-writing. This skill never reads source for issues, never edits code or REVIEW.md.

Flags (always appended together): `--fix` (spawn gsd-code-fixer; default Critical+Warning), `--all` (add Info to fix scope).

vs `gsx-uat-planfix` / `gsx-uat-quickfix`: this audits the changed code (bugs/security/quality); those close acceptance Gaps in VERIFICATION.md / UAT.md.

## 0 — Parse the Phase number

Phase dirs: `.planning/phases/{NN}-{slug}/`, `NN` zero-padded to 2 digits.

- `$ARGUMENTS` has a number → normalize (ignore case, `Phase` prefix, whitespace): `4`/`04`/`Phase 4` → `04`.
- Empty → infer: (1) Phase already under discussion in this conversation; else (2) `.planning/STATE.md` `Last completed phase`; else (3) `AskUserQuestion` (header `Review Phase`) — don't guess.
- No `.planning/` → not a GSD project: tell the user "`/gsd:code-review` cannot run (no `.planning/`)" and stop.
- Pass-through: keep any `--depth=...` / `--files=...`; always append `--fix --all`.

## 1 — Invoke

```
Skill("gsd:code-review", args="{NN} --fix --all")
```

Compose pass-throughs as `"{NN} --depth=deep --fix --all"`, `--fix --all` last. The command runs its full flow (validate → config gate → file scoping → reviewer → REVIEW.md → fixer → summary). Don't interfere while it runs. Then go to step 2.

## 2 — Relay the result

Relay the command's inline summary as a receipt; don't re-judge:

```
## gsx-code-review — Phase {NN}
Command:    /gsd:code-review {NN} --fix --all
Report:     .planning/phases/{NN}-*/{NN}-REVIEW.md
Findings:   Critical {n} · Warning {n} · Info {n}
Fixed:      {fixes applied (+ commits if any)}
Still open: {remaining / not auto-fixable / needs manual confirm}
```

- Incomplete fixes / findings remaining → list outstanding items + next steps (manual / re-run / `/gsd:quick`).
- No changed files / empty scope → report accurately, don't fabricate.
- Commits/branching handled by the command; if fixes left uncommitted in the working tree, remind the user to confirm and commit.

## Guardrails

- Only orchestrate. No reading source for issues, no Edit/Write of any source file or REVIEW.md.
- Flags fixed as `--fix --all`; only pass through user `--depth=`/`--files=`; add no other flags.
- Don't touch STATE.md, don't commit on the command's behalf, don't switch branches.
- One Phase per run. Ambiguous → AskUserQuestion. Non-GSD project → stop.
- Reply in Chinese.
