---
name: gsx-uat-quickfix
description: "Fix acceptance Gaps from both *-VERIFICATION.md (auto-verify; frontmatter status: gaps_found + gaps: list / ### Gaps Summary) and *-UAT.md / *-HUMAN-UAT.md ## Gaps (verify-work) one by one via /gsd:quick: resolve Phase number (inferred from context / STATE.md if blank), Gap name (e.g. GAP-1 / VERIFY-1; all open Gaps if blank) and an optional --discuss flag (run every Gap's /gsd:quick as /gsd:quick --discuss — a lightweight discussion phase before planning), fix each with /gsd:quick, then write the source file(s) back marking them fixed. Invoke when the user says 'fix UAT Gaps', 'fix Phase X's GAP-1', 'gsx-uat-quickfix', 'close Phase X's acceptance gaps', 'fix the UAT Gaps but discuss each one first'. vs gsx-uat-planfix: that batches Gaps via plan-phase + execute-phase (many, complex); this fixes one by one via /gsd:quick (few, scattered, small). vs gsx-quick: that runs /gsd:quick --discuss on one ad-hoc problem the user describes; this drives Gaps recorded in a Phase's acceptance files and writes their status back."
argument-hint: "[optional: Phase number, e.g. 4 / 04 / Phase 4] [optional: Gap name, e.g. GAP-1 / VERIFY-1; all open Gaps if blank] [optional: --discuss]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Skill
  - AskUserQuestion
---

# gsx-uat-quickfix

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Fix acceptance Gaps from both `*-VERIFICATION.md` (`status: gaps_found`) and `*-UAT.md` / `*-HUMAN-UAT.md` (`## Gaps`) one by one via `/gsd:quick`, then write each back as fixed to its respective source file(s).

**Boundary:** This skill only orchestrates (parse Gaps → `/gsd:quick` per Gap → write back to source file(s) → commit the write-back). All code changes go through `/gsd:quick`. The only files this skill writes are the Gap source files (`*-VERIFICATION.md`, `*-UAT.md` / `*-HUMAN-UAT.md`), only after a Gap is fixed by `/gsd:quick` with a commit. Never touch STATE.md, commit **code**, or switch branches — the only thing this skill commits is its own Gap-status write-back, scoped to the source file(s) it edited. Only handles VERIFICATION / UAT Gaps — Gaps in a `{NN}-AUTOTEST.md` (from gsx-uat-autorun) are out of scope; tell the user to fix those directly with `/gsd:quick`.

---

## 0 — Parse args (Phase number + Gap name + --discuss)

`$ARGUMENTS` may have any, all, or none of these:

- **`--discuss` flag**: strip it out first, anywhere in the args, then record `DISCUSS=yes` **literally in the conversation** (the Skill has no variable persistence, and step 4b runs once per Gap — it needs to read the decision back). Absent → `DISCUSS=no`. Everything below parses the args with the flag already removed, so `--discuss` can never be mistaken for a Phase or Gap token.
- **Gap name**: `GAP-1` / `VERIFY-1` / `gap 1` / `verify 1` / `GAP-01` / `gap-2` (ignore case, hyphens, spaces) → `GAP-{n}` or `VERIFY-{n}`.
- **Phase number**: `Phase`-prefixed, or a bare numeric token not matching a Gap name (`4`/`04`/`Phase 4`) → zero-padded `04`. One bare number with no `gap`/`verify` keyword → Phase number.
- Phase blank → infer: (1) Phase under discussion; (2) else STATE.md `Last completed phase`; (3) ambiguous → `AskUserQuestion`, don't guess.
- Gap name blank → fix **all open Gaps** (step 2 filtering).
- No `.planning/` → not a GSD project; tell user and stop.

`--discuss` is a **run-level** switch, not a per-Gap one: it applies to every Gap this run fixes. It changes nothing about how this skill selects, skips, or writes back Gaps — the only thing it does is turn each `/gsd:quick` call in step 4b into `/gsd:quick --discuss`, so that Gap gets a lightweight discussion phase before planning instead of the default fast path. Users reach for it when the Gaps are the kind where the root cause is still fuzzy and a fast blind fix would likely miss. `--discuss` is the only flag this wrapper knows; if the user writes some other `--flag`, don't guess at it — say it isn't recognized and ask whether they meant `--discuss`.

## 1 — Locate Gap source files

File names prefixed with number only (no slug):

```bash
ls .planning/phases/{NN}-*/{NN}-VERIFICATION.md \
   .planning/phases/{NN}-*/{NN}-UAT.md \
   .planning/phases/{NN}-*/{NN}-HUMAN-UAT.md 2>/dev/null
```

Parse both source types (Gaps may be in only one). If both UAT and HUMAN-UAT match, prefer `HUMAN-UAT.md`; note choice in report.

- 0 files → "Phase {NN} has no acceptance output — run `/gsd:execute-phase {NN}` or `/gsd:verify-work {NN}` first." Stop.
- ≥1 match → continue.

## 2 — Parse and merge pending Gaps

### 2a — VERIFICATION file

Check frontmatter `status` first:

- `status: gaps_found` → primary source is the frontmatter `gaps:` YAML list (each entry: `truth` / `status: failed` / `reason` / `artifacts[].path`+`issue`). Extract each as a Gap with a stable id (`VERIFY-1`, `VERIFY-2`); `truth` = title, keep `status`/`reason`/`artifacts`.
- `status: passed` (or no `gaps:`) → no VERIFICATION Gaps.

Then check `### Gaps Summary` prose as supplement (usually mirrors frontmatter). If frontmatter has no `gaps:` but prose exists, extract from prose. "no gap"/"no gaps" → none.

### 2b — UAT file `## Gaps`

Two formats, both handled:

**A. Markdown heading** (Phases 03/04): `### GAP-1：编辑机构无 UI 入口（severity: major）` + optional body. Gap name = `GAP-{n}`; extract title, `severity`, body.

**B. YAML list** (verify-work native):
```yaml
- truth: "<expected>"
  status: open            # or resolved
  reason: "<User reported: ...>"
  severity: major
  test: 3                 # = Test 3 in ## Tests
  root_cause: |
    <root cause>
  artifacts:
    - path: "<file>"
      issue: "<problem / fix note>"
  missing:
    - "<outstanding item>"
```
No `GAP-N`; index by `test: N` ("Test {N}'s Gap").

### 2c — Already closed (skip)

- VERIFICATION: entry `status: passed`/`resolved`, or whole file `status: passed`, or Gap gone from `gaps:`.
- UAT Markdown: title struck `~~`, or `→ **已修复**` / `已修复` / `resolved`.
- UAT YAML: `status: resolved`/`done`/`closed`.

Not matching any of the above = pending fix.

### 2d — Merge

Merge open Gaps from both sources into one pending-fix list. Gaps clearly referring to the same issue (high overlap in `truth`/title/root cause/files) → merge into one, noting source (VERIFICATION / UAT / both).

- Both Gap sections absent/empty → "Phase {NN}'s acceptance output has no Gap records." Stop.
- All Gaps already closed → "All Gaps in Phase {NN} are already closed." Stop.

## 3 — Select Gaps

- **Gap name given** → look up:
  - Found + open → select (single fix).
  - Found + closed → "{Gap name} is already closed — no fix needed." Stop.
  - Not found → list this Phase's Gaps (name + one-liner), ask user to specify; stop.
  - Given `GAP-N` but file uses YAML style (no names) → explain "this UAT uses test-indexed Gaps"; list `Test {N}: {reason}` to pick; stop.
- **Gap name blank** → select all open Gaps.

Show a summary table (Gap name / title / severity / source / Test), and state the mode alongside it so the user sees which pipeline is about to run: `/gsd:quick`(默认档) or `/gsd:quick --discuss`(讨论).

**If >1 Gap selected**, `AskUserQuestion` (header `Fix scope`): "About to fix N Gaps one by one via `/gsd:quick{ --discuss}`, each its own quick-task + commit{（discuss 档：每个 Gap 都会先讨论，耗时略长且会逐个交互提问）}. Start?" → `Start fixing one by one` · `Only fix certain ones (please specify)` · `Cancel`. The `--discuss` cost note is worth stating because the flag multiplies across Gaps — a user who typed `--discuss` for one stubborn Gap may not have pictured it running four times over. It's a heads-up, not a veto: if they confirm, run all N with discussion. Single Gap → step 4 directly.

## 4 — Fix each Gap with /gsd:quick

Process in order (finish one before the next).

### 4a — Suitability

Read all of this Gap, plus its details (VERIFICATION: `truth`, `reason`, `artifacts`; UAT: `expected`, `root_cause`, `missing` from `## Tests` / `## Gaps`). Then:

- **Non-code-defect** — "needs manual follow-up testing" / "depends on data prereq / real SDK / upstream API" / "needs manual user action (restart, reset DB)" / "external env or doc issue, no clear code change point" → **skip**; don't call `/gsd:quick`; record skip reason.
- **Potentially large** — spans subsystems, needs research / multi-file features → still pass to `/gsd:quick`, but warn: if `/gsd:quick` judges scope too large and suggests `/gsd:plan-phase --gaps`, mark that Gap "not fixed (scope exceeds quick)".
- **All others** (clear code/config/doc defect) → 4b.

### 4b — Call /gsd:quick

`DISCUSS=no` (default):
```
Skill("gsd:quick", args="Fix Phase {NN} acceptance Gap '{Gap name or title}' (source: {source file}): {description / reason / root_cause}. Expected behavior: {truth / expected}. Known clues: {artifact paths / missing items / issue}")
```

`DISCUSS=yes` — identical, with `--discuss` leading the args so `/gsd:quick` parses it as a flag rather than as part of the Gap description:
```
Skill("gsd:quick", args="--discuss Fix Phase {NN} acceptance Gap '{Gap name or title}' (source: {source file}): {description / reason / root_cause}. Expected behavior: {truth / expected}. Known clues: {artifact paths / missing items / issue}")
```

Every Gap in the run gets the same treatment — `--discuss` is either on all of them or none, per the step 0 decision. Don't decide per Gap.

`/gsd:quick` runs its full flow (investigate → backend change needs plan confirmation → edit → commit → write `.planning/quick/{id}/` → update STATE.md); under `--discuss` it additionally discusses gray areas with the user before planning, which takes a bit longer and may ask questions per Gap — let it, and don't answer on the user's behalf. On return, record `quick_id` + commit hash, then:
- Fixed with a commit → step 5 to write back this Gap.
- User undid/cancelled, or no commit (no changes) → do NOT write back as fixed; mark "not fixed (cancelled / no changes)"; continue to next.

One Gap at a time.

## 5 — Write back (fixed Gaps only)

After each Gap is fixed (commit produced), `Edit` its source file(s) — only that one entry, today's date.

### 5a — UAT file

**Markdown style** — strike title `~~`, keep `（severity: …）` outside, append `→ **已修复**`, insert note below:
```
### ~~GAP-1：编辑机构无 UI 入口~~（severity: major）→ **已修复**
**修复：** quick-task {quick_id}，commit {hash}（{YYYY-MM-DD}）。{one sentence}
```

**YAML style** — set `status: resolved`, add below:
```
  status: resolved
  fixed_by: "quick-task {quick_id}, commit {hash} ({YYYY-MM-DD})"
```

If `## Summary` has `open_gaps:`, decrease by the number fixed (not below 0).

### 5b — VERIFICATION file

If the Gap came from or appears in `*-VERIFICATION.md`:
1. In frontmatter `gaps:` YAML list, set entry `status: failed` → `resolved` (or `passed`), add `fixed_by: "quick-task {quick_id}, commit {hash} ({YYYY-MM-DD})"`.
2. In `### Gaps Summary` prose, append `→ **已修复**（quick-task {quick_id}, commit {hash}, {YYYY-MM-DD}）` after the matching Gap description.

Then commit the write-back. The write-back is its own change (`/gsd:quick` already committed the code + quick docs per Gap; the Gap status write-back belongs to no single quick-task), so give it one atomic commit covering this run's write-backs:

```bash
git add .planning/phases/{NN}-*/{NN}-*UAT.md .planning/phases/{NN}-*/{NN}-VERIFICATION.md 2>/dev/null
git commit -m "docs({NN}): acceptance Gap 修复回写（{fixed gap names}）"
```

Scope `git add` to the exact source file path(s) this skill edited — never `git add -A` / `git add .`. Concurrent GSD work may have unrelated files staged; this skill must commit **only** the acceptance write-back, never code or anything else. Commit once at the end of the run, after all selected Gaps are written back — not per Gap. If nothing was written back as fixed (all skipped / cancelled / no-change), there is nothing to commit — skip it. Record the commit hash in the report.

## 6 — Summary report

```
## gsx-uat-quickfix — Phase {NN}
Sources: {VERIFICATION filename (has/no Gaps)} + {UAT filename (has/no Gaps)}

Mode: /gsd:quick{ --discuss}

Fixed:
- {Gap name} "{title}" (source: {VERIFICATION/UAT}) → quick-task {quick_id}, commit {hash}

Skipped / not fixed:
- {Gap name} "{title}" — {reason: non-code-defect / cancelled / scope exceeds quick…}

Write-back: {UAT filename / VERIFICATION filename} marked {N} Gaps fixed. Committed as docs({NN}) {write-back commit hash}. (If no Gap was fixed, no write-back / commit was made.)
```

For skipped/not-fixed items, give each a reason + next step (manual follow-up / `/gsd:plan-phase --gaps` / re-run this skill).

## Guardrails

- All code changes go 100% through `/gsd:quick`; this skill writes — and commits — only the acceptance source files (`*-VERIFICATION.md`, `*-UAT.md` / `*-HUMAN-UAT.md`), only after a confirmed commit. Cancellations / no-change outcomes are never written back as fixed. The write-back commit is scoped to the source file paths alone (never `git add -A`), so it can never sweep up code or other staged files.
- Skip already-closed Gaps (`~~struck~~` / `status: resolved` / `status: passed`).
- One Phase per run. One Gap = one `/gsd:quick` call, in series — never bundle multiple Gaps into one. `--discuss` doesn't change that shape: it's still one Gap per call, just each call discusses first.
- `--discuss` only ever reaches `/gsd:quick` as a flag on its args — never treat it as a reason to do extra investigation, research, or verification **here**. This skill's boundary (orchestrate + write back acceptance files) is the same in both modes; the discussion phase belongs to `/gsd:quick`.
- When unsure a Gap is a code defect, prefer skip-with-note over forcing it into `/gsd:quick`.
- Reply in Chinese.
