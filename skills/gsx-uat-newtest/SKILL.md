---
name: gsx-uat-newtest
description: "Append one new Test to the `## Tests` section of a GSD Phase's HUMAN-UAT.md / UAT.md (produced by /gsd:verify-work): parse the user's Test description, fill missing fields (title / expected / result) in one AskUserQuestion pass; auto-continue Test numbering from max N (never reused), follow the file's existing single- or multi-line `expected:` style, and bump frontmatter `updated:` + `## Summary` (`total:` +1 and the matching result count pending/passed/issues/skipped/blocked +1). Touches only the UAT file — no source, no STATE.md — then commits that one UAT file (scoped to its path). Sibling of gsx-uat-newgap (registers a Gap, not a Test). Invoke explicitly when the user says 'add a Test', 'new-test', 'gsx-uat-newtest', 'log this acceptance point as a Test', 'add another test case'."
argument-hint: "[required: Test description, free text, with at least a title or expected behavior] [optional: Phase number, e.g. 4 / 04 / Phase 4; inferred from context / STATE.md if blank]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - AskUserQuestion
---

# gsx-uat-newtest

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Append **one** new Test to the `## Tests` section of a Phase's HUMAN-UAT.md / UAT.md, bump `updated:` + `## Summary`, then commit the UAT file. Only touches the UAT file — no source, no STATE.md, no execution; the only thing it commits is that single UAT file. To run tests use `gsx-uat-autorun` / `/gsd:verify-work`.

## 0 — Parse args (Test description + optional Phase number)

`$ARGUMENTS` is free text (description and/or Phase number, or neither).

- **Phase number**: `Phase`-prefixed or bare 1–2 digit token (`4` / `04` / `Phase 4`), or a decimal inserted phase (`10.1`). Keep decimals as-is; pad bare integers to 2 digits. Multiple candidates / ambiguous → don't guess, `AskUserQuestion`.
- **Test description**: remaining text → raw material for step 3.

Phase number blank → infer: (1) Phase under discussion in this conversation; (2) else `.planning/STATE.md` `Last completed phase`; (3) still not unique → `AskUserQuestion` (header `Phase`).

Test description empty → `AskUserQuestion` (header `Test description`) for one sentence covering "what to test / what to expect".

No `.planning/` → not a GSD project; tell user and stop.

## 1 — Locate the UAT file

```bash
ls .planning/phases/{NN}-*/{NN}-*UAT.md
```

- 0 matches → stop; tell user "Phase {NN} has no UAT.md / HUMAN-UAT.md; run `/gsd:verify-work {NN}` first. This skill won't generate the file."
- Both `{NN}-UAT.md` + `{NN}-HUMAN-UAT.md` → prefer `HUMAN-UAT.md`; note choice in report.
- 1 match → target.

## 2 — Detect existing format + next number

Read content under `## Tests`; match the new Test to the existing style.

Standard structure:

```
### N. <title>
expected: <expected>          # single-line; or `expected: |` + 2-space-indented block for multi-line
result: pass | issue | pending | skipped | blocked
note: <evidence>              # optional, only when result != pending
```

- Title anchor `### N. <title>` (trailing English period).
- `expected:` single vs multi: grep existing for `expected: |`.
- `result:` set: `pass / issue / pending / skipped / blocked` (some files also `partial`). New Test defaults `pending`.

Rules:
- `## Tests` has `### N.`+`expected:` entries → follow that structure + single/multi-line style.
- `## Tests` absent/empty → default structured single-line; first Test is Test 1 (step 5 adds the heading if missing).
- **Free-form report style** (`### 2.1 起栈`-type headings, no `### N.`+`expected:`+`result:`, e.g. Phase 14) → fallback: do NOT force the template; tell user "free-form report UAT — append manually or migrate to structured first" and stop.

Next number: grep all `### N.` lines, take max integer N, new = N+1. **All Tests (any result) count; numbers never reused, monotonic in time order.**

## 3 — Smart-parse + prompt for missing required fields

Extract from the description (mark unknowns "ask"):

| Field | Clues | Required? |
|-------|-------|-----------|
| `title` | one-sentence "what object + what action" | Required (ask if missing) |
| `expected` | expected/observable acceptance point ("expected:", "期望：", "should see") | Required (ask if missing) |
| `result` | "pass", "already pass", "pending", "failed issue" | Default **`pending`** if missing |
| `note` | actual evidence; only meaningful when result != pending | Optional |

One pass: pack ALL missing required fields into a **single** `AskUserQuestion` (max 4 questions, required-priority order). Don't ask per-field; if all present, don't ask.

`result` candidates (fixed): `pending (not yet run) (Recommended)` / `pass` / `issue (found a problem)` / `skipped (can't determine)` / `blocked (env/dependency unavailable)`.

When asking `expected`, push observable assertions (e.g. "endpoint X returns 200 + R.data has vcId", "/console/users shows paginated table w/ column Y") not "should work"; give a short example in the question.

## 4 — Duplicate-registration warning

Before writing, compare extracted `title` against existing `### N. <title>` headings. Significant overlap (≥3 effective word tokens vs one entry) → `AskUserQuestion` (header `Duplicate registration`):
- `New Test — continue (Recommended)` → step 5.
- `Supplement to Test X` → do NOT create a new entry; append a "**Supplement ({YYYY-MM-DD}):** …" line under that Test's `note:` (insert `note:` if absent). Then step 6; **Summary unchanged** (no new Test, counts unchanged).
- `Don't register` → stop.

## 5 — Write the new Test

`Edit`-append at the **end** of `## Tests` (just before the next `## ` heading, or EOF). One blank line before/after. If `## Tests` absent → add heading after frontmatter, before `## Summary`. Chinese Markdown rules (space between Chinese & numbers; Chinese punctuation).

Single-line template:
```
### {N}. {title}
expected: {expected}
result: {result}
```
Multi-line template:
```
### {N}. {title}
expected: |
  {line 1}
  {line 2}
result: {result}
```
(2-space indent, consistent with file). When result != pending and user gave a note → add `note: {note}` (single) or `note: |` block.

Single vs multi selection:
- single sentence, <80 chars, no newlines → follow majority style (multi-line if >50% existing use `expected: |`, else single).
- contains newlines or >80 chars → force multi-line.
- `## Tests` empty → default single-line.

## 6 — Bump frontmatter `updated:`

In the `---` frontmatter, set `updated:` to current ISO-8601 UTC to the second (`date -u +"%Y-%m-%dT%H:%M:%SZ"`, never hardcode).
- exists → Edit value.
- absent but frontmatter exists → insert below `started:`.
- no frontmatter → don't force-add; note in report.

## 7 — Update `## Summary` counts

- `total:` → +1; absent → insert `total: 1` at start of Summary.
- matching result count +1 (insert `= 1` if absent): `pending`→`pending:`, `pass`→`passed:`, `issue`→`issues:`, `skipped`→`skipped:`, `blocked`→`blocked:`. Note name shifts: `pass`→`passed:` (past tense), `issue`→`issues:` (plural).
- absent field but `total:` exists → insert keeping order (`total / passed / issues / pending / partial / skipped / blocked / open_gaps`).
- preserve inline comments; increment number and append Test ref in comment (e.g. `pending: 2  # Test 4 待跑；Test {N}（新增）`).
- `## Summary` absent → don't force-add; note in report.
- **Step-4 "supplement" branch → `total:` and all counts unchanged.**

## 7b — Commit the UAT file

Commit the edit — only the UAT file. Registering a Test is a standalone doc change, so give it its own atomic commit:

```bash
git add .planning/phases/{NN}-*/{NN}-*UAT.md
git commit -m "docs({NN}): UAT 新增 Test {N}（{title}）"
```

Scope `git add` to the exact UAT file path — never `git add -A` / `git add .`. Concurrent GSD work may have unrelated files staged; this skill commits **only** the UAT file, nothing else. This also covers the step-4 "supplement" branch (a supplement line was still written — there use message `docs({NN}): UAT Test {X} 补充说明`). Only skip the commit when nothing was written at all (step-4 "Don't register" → stop). Record the commit hash in the report.

## 8 — Summary report

```
## gsx-uat-newtest — Phase {NN} ({filename})
New Test registered:
- Test {N}: {title} · expected: {summary} · result: {result} · note: {summary or (none)}
Updated: `## Tests` appended · frontmatter `updated:` → {ts} · `## Summary` `total:` {old}→{new}, `{field}:` {old}→{new}
Committed as docs({NN}) {write-back commit hash}.
Next: gsx-uat-autorun {NN} (auto-run) · /gsd:verify-work {NN} (manual) · gsx-uat-newgap {NN} (register a Gap)
```

## Guardrails

- Only the UAT file (`*-UAT.md` / `*-HUMAN-UAT.md`); never source / STATE.md / SUMMARY / `.planning/quick/`. The commit is scoped to the UAT file path alone (never `git add -A`), so it can never sweep up source or other staged files.
- No branch switch, no execution/verification. The only commit it makes is the UAT write-back itself.
- Numbers only increase, never reused (all results count): new = max(N)+1.
- Follow existing format, don't enforce: single stays single, multi stays multi; free-form report → fallback stop.
- Warn on duplicates; leave the decision to the user.
- One Test per run — re-invoke for more.
- Reply in Chinese with Chinese Markdown formatting.
