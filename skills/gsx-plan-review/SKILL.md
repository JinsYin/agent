---
name: gsx-plan-review
description: "Thin front-door to /gsd:review --phase N --codex — gets an independent cross-AI peer review of a GSD Phase's PLAN.md from the external Codex CLI, producing REVIEWS.md. Use after a phase is planned (PLAN.md exists) and you want a second pair of eyes before executing — 'review the plan with Codex', 'peer-review Phase 3's plan', 'cross-AI review', 'get Codex to critique this phase plan', 'plan-review', 'gsx-plan-review', or just a phase number in a post-plan / pre-execute context. Arg is the Phase number (inferred from context / STATE.md last planned phase if blank). When the review lands it recommends /gsx-replan-phase to fold the feedback back into the plan. NOT code review (that's gsx-code-review, which audits implemented source) — this reviews the PLAN.md before any code is written."
argument-hint: "[optional: Phase number, e.g. 4 / 04 / Phase 4; inferred from context or STATE.md if blank]"
allowed-tools:
  - Read
  - Glob
  - Bash
  - Skill
  - AskUserQuestion
---

# gsx-plan-review

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Thin orchestrator over `/gsd:review`: parse the Phase number → call `/gsd:review --phase {N} --codex` → relay the result and point the user at `/gsx-replan-phase`. The command owns everything — detecting the Codex CLI, building the review prompt from `PLAN.md`, invoking Codex, collecting the response, and writing `REVIEWS.md`. This skill never reviews the plan itself and never edits any file.

**Why Codex specifically:** an independent external model reading the plan cold catches what the planner (and you, having just written it) are blind to — unstated assumptions, missing edge cases, a sequencing flaw, an over-scoped task. The point of a cross-AI pass is *divergent* judgment before you commit the plan to execution, where mistakes get expensive. This skill fixes the reviewer to `--codex`; if you want a different or wider panel (`--gemini`, `--all`, …), call `/gsd:review` directly.

**Boundary — this skill never changes the plan.** It produces `REVIEWS.md` (the feedback), nothing more. Folding that feedback back into `PLAN.md` is a separate, deliberate step — that's `/gsx-replan-phase` (which runs `/gsd:plan-phase N --reviews`). Keeping review and replan separate means you get to read the critique and decide before the plan changes. Reply in Chinese.

---

## 0 — Parse the Phase number

Phase dirs: `.planning/phases/{NN}-{slug}/`, `NN` zero-padded to 2 digits. `/gsd:review` needs a planned phase — a `PLAN.md` must already exist (from `gsx-plan-phase` / `/gsd:plan-phase`).

- `$ARGUMENTS` has a number → normalize (ignore case, `Phase` prefix, whitespace): `4`/`04`/`Phase 4` → `4`.
- Empty → infer: (1) Phase already under discussion in this conversation; else (2) `.planning/STATE.md` last planned / last completed phase; else (3) `AskUserQuestion` (header `Review Phase`) — don't guess.
- No `.planning/` → not a GSD project: tell the user "`/gsd:review` cannot run (no `.planning/`)" and stop.
- No `PLAN.md` for the phase yet → there's nothing to review: tell the user to plan first (`/gsx-plan-phase {N}`) and stop.

## 1 — Invoke

```
Skill("gsd:review", args="--phase {N} --codex")
```

The command runs its full flow (detect Codex CLI → build prompt from `PLAN.md` → invoke Codex → collect response → write `REVIEWS.md`). Don't interfere while it runs. If the Codex CLI isn't installed/detected, the command will say so — relay that plainly; don't substitute another reviewer silently. Then go to step 2.

## 2 — Relay the result + recommend the next step

Relay the command's outcome as a receipt; don't re-judge the plan yourself:

```
## gsx-plan-review — Phase {N}
Command:  /gsd:review --phase {N} --codex
Reviewer: Codex CLI
Report:   .planning/phases/{NN}-*/{NN}-REVIEWS.md
Verdict:  {one line — Codex's headline take, e.g. HIGH concerns raised / minor suggestions / looks solid}
Concerns: {count by severity if the report classifies them, else a one-line gist}
```

Then **always** recommend the next step — this is the whole point of producing `REVIEWS.md`:

```
下一步：/gsx-replan-phase {N} — 把本次 Codex 评审意见（REVIEWS.md）回灌进计划，重新生成 PLAN.md
```

- If Codex raised real concerns (especially HIGH), make the recommendation emphatic — the plan shouldn't go to execution with open HIGH concerns.
- If Codex found nothing material ("looks solid"), still mention `/gsx-replan-phase` is available but note replanning may be optional — the user can go straight to `/gsd:execute-phase {N}`.
- If the Codex CLI was unavailable / the run produced no `REVIEWS.md`, say so plainly and name what to do (install/configure Codex, or use `/gsd:review --phase {N} --all` for another reviewer).

---

## Guardrails

- **Only orchestrate.** No reading `PLAN.md` to review it yourself, no Edit/Write of any plan or `REVIEWS.md`. The command and Codex own the review.
- **Reviewer is fixed to `--codex`.** Don't silently swap in another CLI. If Codex is unavailable, report it; switching panels is the user's call (`/gsd:review` directly).
- **Always surface `/gsx-replan-phase` as the next step** — `REVIEWS.md` exists to be folded back into the plan; leaving it unconsumed wastes the review.
- **Don't replan here.** This skill stops at `REVIEWS.md`. Folding feedback into `PLAN.md` is `/gsx-replan-phase`'s job — kept separate so the user reads the critique first.
- **Needs a planned phase.** No `PLAN.md` → nothing to review → send the user to `/gsx-plan-phase` first.
- **Not code review.** This reviews the *plan* pre-execution; auditing implemented source is `/gsx-code-review`.
- **One Phase per run.** Ambiguous → `AskUserQuestion`. Non-GSD project → stop.
- Reply in Chinese.
