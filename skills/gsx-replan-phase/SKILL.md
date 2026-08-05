---
name: gsx-replan-phase
description: "Thin front-door to /gsd:plan-phase N --reviews — replans a GSD Phase by folding cross-AI review feedback (REVIEWS.md, produced by /gsd:review or /gsx-plan-review) back into PLAN.md. Use after a plan has been peer-reviewed and you want the critique incorporated — 'replan with the review feedback', 'fold Codex's comments into the plan', 'redo Phase 3's plan addressing the reviews', 'incorporate REVIEWS.md', 'replan-phase', 'gsx-replan-phase', or just a phase number right after a /gsx-plan-review run. Arg is the Phase number (inferred from context / STATE.md if blank). This is the natural follow-up to /gsx-plan-review. NOT the same as gsx-plan-phase (fresh research+plan from scratch) — this revises an existing plan against existing review feedback, so it needs a REVIEWS.md to consume."
argument-hint: "[optional: Phase number, e.g. 4 / 04 / Phase 4; inferred from context or STATE.md if blank] [optional plan-phase passthrough flags: --tdd --mvp --skip-verify --text ...]"
allowed-tools:
  - Read
  - Glob
  - Bash
  - Skill
  - AskUserQuestion
---

# gsx-replan-phase

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Thin orchestrator over `/gsd:plan-phase`: parse the Phase number → call `/gsd:plan-phase {N} --reviews` → relay the result. The `--reviews` flag tells `plan-phase` to read the phase's `REVIEWS.md` (the cross-AI peer-review feedback) and re-run the planner so the regenerated `PLAN.md` answers the reviewers' concerns. The command owns the replanning — spawning `gsd-planner`, the `gsd-plan-checker` verify loop, and the `PLAN.md` write. This skill never drafts or edits the plan itself.

**Why it exists / where it sits:** this is the third beat of the plan→review→replan convergence loop. `gsx-plan-phase` produces the first `PLAN.md` (docs-grounded). `gsx-plan-review` runs `/gsd:review --phase N --codex` and writes `REVIEWS.md`. This skill closes the loop by feeding that critique back in. The value is that the regenerated plan is shaped by an *independent* reviewer's judgment, not just the original planner's — assumptions get challenged, gaps get filled, before any code is written. The loop can repeat: replan → review again → replan, until no HIGH concerns remain (cf. `/gsd:plan-review-convergence`).

**Boundary — this skill never changes code.** It revises `PLAN.md` (via `/gsd:plan-phase --reviews`) against existing `REVIEWS.md`; it does not edit source, run migrations, or commit beyond what `plan-phase` does. Implementation belongs to `/gsd:execute-phase` afterward. Reply in Chinese.

---

## 0 — Parse the Phase number (+ pass-through flags)

Phase dirs: `.planning/phases/{NN}-{slug}/`, `NN` zero-padded to 2 digits. `--reviews` needs the phase's `REVIEWS.md` to already exist (from `/gsx-plan-review` / `/gsd:review`).

- `$ARGUMENTS` has a number → normalize (ignore case, `Phase` prefix, whitespace): `4`/`04`/`Phase 4` → `4`. Keep any trailing `plan-phase` flags (`--tdd`, `--mvp`, `--skip-verify`, `--text`, …) — they pass through unchanged.
- Empty → infer: (1) Phase already under discussion in this conversation (e.g. a `/gsx-plan-review` just finished); else (2) `.planning/STATE.md` last planned / last reviewed phase; else (3) `AskUserQuestion` (header `Replan Phase`) — don't guess.
- No `.planning/` → not a GSD project: tell the user "`/gsd:plan-phase` cannot run (no `.planning/`)" and stop.
- No `REVIEWS.md` for the phase yet → there's no feedback to fold in: tell the user to run `/gsx-plan-review {N}` first (or, for a fresh plan, `/gsx-plan-phase {N}`) and stop. Don't run `--reviews` against a missing file.

## 1 — Invoke

```
Skill("gsd:plan-phase", args="{N} --reviews {any pass-through flags}")
```

`--reviews` makes `plan-phase` consume `REVIEWS.md` and replan; it does not re-research, so the grounded `RESEARCH.md` from `gsx-plan-phase` stays intact. Let the command own the planner spawn, the `gsd-plan-checker` verify loop, iteration to pass, and the `PLAN.md` rewrite. Don't pre-empt it by drafting tasks yourself. Then go to step 2.

## 2 — Relay the result

Relay the command's outcome as a receipt; don't re-summarize the whole plan:

```
## gsx-replan-phase — Phase {N}
Command:  /gsd:plan-phase {N} --reviews
Input:    .planning/phases/{NN}-*/{NN}-REVIEWS.md (Codex 评审意见)
Output:   .planning/phases/{NN}-*/{NN}-PLAN.md (已按评审意见重生成)
Addressed:{one line — which concerns the replan folded in / any it explicitly deferred}
下一步：/gsd:execute-phase {N}（若仍有未消化的评审意见，可再跑一轮 /gsx-plan-review 复评）
```

- If concerns remain unresolved or the reviewer's points conflict with locked decisions, name that plainly rather than implying a clean sweep — the user may want another review round or a manual call.
- If the verify loop didn't converge or the run bailed before `PLAN.md` was rewritten, say so and name what state the plan is in (old `PLAN.md` unchanged, or partially written).

---

## Guardrails

- **Only orchestrate.** No drafting or editing `PLAN.md` / `REVIEWS.md` yourself. `plan-phase` and its planner own the replan.
- **Needs `REVIEWS.md`.** `--reviews` consumes existing review feedback — no `REVIEWS.md` → send the user to `/gsx-plan-review {N}` first. Don't run `--reviews` against nothing.
- **`--reviews` doesn't re-research.** The grounded `RESEARCH.md` stays as-is; this is a plan revision, not a fresh research+plan. For a from-scratch plan use `/gsx-plan-phase`.
- **Don't review here.** Generating the critique is `/gsx-plan-review`'s job; this skill only folds an existing critique back in. The loop is: plan → review → replan (→ review → replan …).
- **No code changes here.** This is a planning skill. Implementation belongs to `/gsd:execute-phase` afterward.
- **One Phase per run.** Ambiguous → `AskUserQuestion`. Non-GSD project → stop.
- Reply in Chinese.
