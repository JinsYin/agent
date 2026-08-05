---
name: gsx-uat-newgap
description: "Append one new Gap to the `## Gaps` section of a GSD Phase's HUMAN-UAT.md / UAT.md (produced by /gsd:verify-work): parse the user's Gap description, fill missing fields (severity / linked Test number / artifacts) in one AskUserQuestion pass; auto-continue numbering from max GAP-N (following the file's existing Markdown-heading or YAML-list style), and bump frontmatter `updated:` + Summary `open_gaps:` (+1). Touches only the UAT file — no source, no STATE.md — then commits that one UAT file (scoped to its path). Sibling of gsx-uat-newtest (registers a Test, not a Gap). Invoke explicitly when the user says 'add a Gap', 'new-gap', 'gsx-uat-newgap', 'log this issue as a Gap in Phase X's UAT'."
argument-hint: "[required: Gap description, free text] [optional: Phase number, e.g. 4 / 04 / Phase 4; inferred from context / STATE.md if blank]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - AskUserQuestion
---

# gsx-uat-newgap

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Append **one** new Gap to the `## Gaps` section of a Phase's HUMAN-UAT.md / UAT.md, bump `updated:` + `## Summary` `open_gaps:`, then commit the UAT file. Only touches the UAT file — no source, no STATE.md, no diagnosis/fixing; the only thing it commits is that single UAT file. To fix use `gsx-uat-quickfix` / `gsx-uat-planfix`.

## 0 — Parse args (Gap description + optional Phase number)

`$ARGUMENTS` is free text (description and/or Phase number, or neither).

- **Phase number**: `Phase`-prefixed or bare 1–2 digit token (`4` / `04` / `Phase 4`); zero-pad to 2 digits. Multiple candidates / ambiguous → don't guess, `AskUserQuestion`.
- **Gap description**: remaining text → raw material for step 3.

Phase number blank → infer: (1) Phase under discussion; (2) else `.planning/STATE.md` `Last completed phase`; (3) still not unique → `AskUserQuestion` (header `Phase`).

Gap description empty → `AskUserQuestion` (header `Gap description`) for one sentence covering "expected / current state / impact".

No `.planning/` → not a GSD project; tell user and stop.

## 0b — Detect whether the description should split into multiple Gaps

If the Gap description shows signs of bundling independent problems (numbered list `1) 2) 3)`; enumerated `一、二、三`; or clauses joined by "另外" / "还有" / "以及" / "再者" that each point at a different module / file / symptom), don't default to splitting and don't default to merging — confirm with the user first via `AskUserQuestion` (header `Split?`):

- `Merge into one Gap (Recommended if same root cause)` — treat the whole description as a single Gap and continue with the single-Gap flow below.
- `Split into {N} separate Gaps` — first restate each sub-problem you identified (one sentence each) so the user can confirm the boundaries; once confirmed, run steps 2–7 once per sub-problem (own numbering, own `AskUserQuestion` field-fill pass, own write), then fold everything into **one shared commit** at step 7b.

Skip this question when the signal is weak (single symptom, single file, single causal chain) — fall through to the normal single-Gap flow.

## 1 — Locate the UAT file

```bash
ls .planning/phases/{NN}-*/{NN}-*UAT.md
```

- 0 matches → stop; tell user "Phase {NN} has no UAT.md / HUMAN-UAT.md; run `/gsd:verify-work {NN}` first. This skill won't generate the file."
- Both `{NN}-UAT.md` + `{NN}-HUMAN-UAT.md` → prefer `HUMAN-UAT.md`; note choice in report.
- 1 match → target.

## 2 — Detect existing format + next number

Read content under `## Gaps`; match the new Gap to the existing style.

**A. Markdown heading style** (Phases 03/04/10):
```
### GAP-1：编辑机构无 UI 入口（severity: major）
<optional body: root cause, files, follow-up...>
```
Identifier: `### GAP-N` or `### ~~GAP-N~~` (closed/struck-through).

**B. YAML list style** (Phases 02/08):
```
- truth: "<expected statement>"
  status: open            # or resolved
  reason: "<User reported: ...>"
  severity: major
  test: 3
  artifacts:
    - path: "<file>"
      issue: "<problem>"
  missing:
    - "<outstanding item>"
```
Identifier: top-level `-` items with `truth:` / `status:` / `severity:`.

Rules:
- any Markdown entries → Markdown style.
- only YAML entries → YAML style.
- both present (rare) → Markdown style; note "mixed formats" in report.
- `## Gaps` absent/empty → default Markdown; first is `GAP-1` (step 5 adds the heading if missing).

Next number:
- Markdown: grep `### GAP-{N}` / `### ~~GAP-{N}~~`, max N, new = N+1. **Closed/struck-through GAPs count; numbers never reused, monotonic in time order.**
- YAML: no explicit name; new gets a `# GAP-{N+1}` comment where N = current list-item count (incl. resolved); `test:` = user-specified test number.

## 3 — Smart-parse + prompt for missing required fields

Extract from the description (mark unknowns "ask"):

| Field | Clues | Required? |
|-------|-------|-----------|
| `title` / `truth` | one-sentence "expected behavior / what should happen" | Required (ask if missing) |
| `reason` | current state / "User reported: …" / "Autotest: …" | Required (fallback to `title`) |
| `severity` | `blocker/major/minor/trivial`, or 阻塞/严重/小问题 | Required (ask if missing, default `major`) |
| `test` | `Test 5`, "relates to Test 5"; or quotes a Test's expected | Required (ask; `0`/`-` for none) |
| `artifacts` | file-path tokens (`/`, `.tsx`/`.java`/`.sql`/`.md`); may be multiple | Optional |
| `missing` | "need to…" / "supplement…" / "missing…" / "TODO" | Optional |
| `notes` | other supplementary info | Optional |

One pass: pack ALL missing required fields into a **single** `AskUserQuestion` (max 4 questions, required-priority order). Don't ask per-field; if all present, don't ask.

`severity` candidates (fixed): `blocker (blocks main flow)` / `major (functional, has workaround) (Recommended)` / `minor (experience/edge-case)` / `trivial (copy/visual)`.

For `test`: first read `## Tests` and list test titles for the user to pick (e.g. "1. Cold-start login / 2. Insured user list … / 0. Not linked").

## 4 — Duplicate-registration warning

Before writing, compare extracted `title`/`truth` keywords against existing Gap titles / `truth:`. Significant overlap (≥3 effective word tokens vs one entry) → `AskUserQuestion` (header `Duplicate registration`):
- `New Gap — continue (Recommended)` → step 5.
- `Supplement to GAP-X` → do NOT create a new entry; append a "**Supplement ({YYYY-MM-DD}):** …" paragraph under that entry. Then step 6; **`open_gaps:` unchanged**.
- `Don't register` → stop.

## 5 — Write the new Gap

`Edit`-append at the **end** of `## Gaps` (just before the next `## ` heading, or EOF). One blank line before/after. If `## Gaps` absent → add heading after `## Summary`. Chinese Markdown rules (space between Chinese & numbers; Chinese punctuation; no Jinja `{}` placeholders in YAML values containing Chinese).

**Markdown heading template:**
```
### GAP-{N}：{title}（severity: {severity}）
**现状：** {reason}
**关联 Test：** Test {test}{if 0 write "（无对应 Test）"}
**涉及文件：**
- `{artifacts[i].path}` —— {artifacts[i].issue}
**待补：**
- {missing[i]}
**备注：** {notes}
```
Omit entire blocks for missing fields (no empty lines / "none" placeholders).

**YAML list template:**
```
- # GAP-{N}
  truth: "{title}"
  status: open
  reason: "{reason}"
  severity: {severity}
  test: {test}
  artifacts:
    - path: "{artifacts[i].path}"
      issue: "{artifacts[i].issue}"
  missing:
    - "{missing[i]}"
  notes: "{notes}"
```
Omit entire fields when artifacts/missing/notes empty — never write `artifacts: []` / `missing: []` placeholders.

## 6 — Bump frontmatter `updated:`

In the `---` frontmatter, set `updated:` to current ISO-8601 UTC to the second (`date -u +"%Y-%m-%dT%H:%M:%SZ"`, never hardcode).
- exists → Edit value.
- absent but frontmatter exists → insert below `started:`.
- no frontmatter → don't force-add; note in report.

## 7 — Update `## Summary` `open_gaps:`

- exists (number) → +1; preserve inline comments, increment and append Gap ref (e.g. `open_gaps: 2  # GAP-2 待修；GAP-3（新登记）`).
- absent → insert `open_gaps: 1` below `blocked:`.
- `## Summary` absent → don't force-add; note in report.
- **Step-4 "supplement" branch → `open_gaps:` unchanged.**

## 7b — Commit the UAT file

Commit the edit — only the UAT file. Registering a Gap is a standalone doc change, so give it its own atomic commit:

```bash
git add .planning/phases/{NN}-*/{NN}-*UAT.md
git commit -m "docs({NN}): UAT 新增 GAP-{N}（{title}）"
```

Scope `git add` to the exact UAT file path — never `git add -A` / `git add .`. Concurrent GSD work may have unrelated files staged; this skill commits **only** the UAT file, nothing else. This also covers the step-4 "supplement" branch (a supplement paragraph was still written — there use message `docs({NN}): UAT GAP-{X} 补充说明`). Only skip the commit when nothing was written at all (step-4 "Don't register" → stop). Record the commit hash in the report.

Step-0b split branch → all sub-Gaps land in one commit: `docs({NN}): UAT 新增 GAP-{N}~GAP-{M}（{count} 项）`.

## 8 — Summary report

```
## gsx-uat-newgap — Phase {NN} ({filename})
New Gap(s) registered:
- GAP-{N}: {title} · severity: {severity} · linked Test: {test} · files: {count} · outstanding: {count}
- (repeat one line per sub-Gap if step-0b split was confirmed)
Updated: `## Gaps` appended · frontmatter `updated:` → {ts} · `## Summary` `open_gaps:` {old}→{new}
Committed as docs({NN}) {write-back commit hash}.
Next: gsx-uat-quickfix {NN} GAP-{N} (quick fix) · gsx-uat-planfix {NN} (batch fix)
```

## Guardrails

- Only the UAT file (`*-UAT.md` / `*-HUMAN-UAT.md`); never source / STATE.md / SUMMARY / `.planning/quick/`. The commit is scoped to the UAT file path alone (never `git add -A`), so it can never sweep up source or other staged files.
- No branch switch, no diagnosis/fixing (use gsx-uat-quickfix / gsx-uat-planfix / /gsd:debug). The only commit it makes is the UAT write-back itself.
- Numbers only increase, never reused (closed/struck-through GAP-N count): new = max(N)+1.
- Follow existing format, don't enforce: Markdown stays Markdown, YAML stays YAML.
- Warn on duplicates; leave the decision to the user.
- One Gap per run by default — re-invoke for more. Exception: step 0b, when the user explicitly confirms a description should be split into multiple Gaps, registers them all in the same run (one shared commit). Never split or merge without asking first.
- Reply in Chinese with Chinese Markdown formatting.
