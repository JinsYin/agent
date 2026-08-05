---
name: gsx-fast
description: "Multi-round-iterate-then-final-squash fast workflow for one trivial change (lighter than gsx-quick: /gsd:fast runs inline — no subagents, no PLAN/SUMMARY, no .planning/quick/{id}/ dir): call /gsd:fast to do the one-liner inline, ask via AskUserQuestion whether to keep tweaking; if yes call /gsd:fast again (no resume — each round is independent inline) until satisfied; finally squash all commits into 1 (subject from the task description) and append one row to the 'Quick Tasks Completed' table in .planning/STATE.md. If not trivial (>3 edits, needs research/planning, multi-file) or touches API return values / how values render (field localization, enum mapping, formatting, masking) → escalate to gsx-quick (it runs the full quick pipeline, which discusses the change before making it). Arg is the problem description; ask if blank. Invoke when the user says 'fast-fast', 'gsx-fast', 'run fast then squash'. vs gsx-quick: that runs /gsd:quick --discuss (discussion + research + planning + plan-checking + verification, PLAN/SUMMARY/task_dir) for weightier problems; this uses /gsd:fast (inline, zero planning) for trivial only."
argument-hint: "[problem description]"
allowed-tools:
  - Read
  - Edit
  - Grep
  - Glob
  - Bash
  - Skill
  - AskUserQuestion
---

# gsx-fast

## Codex Adapter

In Codex, translate this Claude wrapper instead of rewriting it:

- `$ARGUMENTS` = text after `$gsx-*`.
- `/gsd:<name> ...` or `Skill("gsd:<name>", args="...")` = run `.codex/skills/gsd-<name>/SKILL.md` with args as `{{GSD_ARGS}}`.
- `AskUserQuestion`: only map to `request_user_input` if your Codex build truly renders an interactive picker; otherwise (the usual Codex case) do **not** force it. Before any choice among options/plans, first print each option's core content in your reply, then list the choices as a numbered Markdown list and **stop and wait** for the user's text reply. Never self-answer a prompt the user never saw.
- Claude tool names are intent labels; use equivalent Codex tools.
- Preserve wrapper boundaries; if GSD owns edits, do not edit source here.


Lightest quick-family tier. Closed loop for a **trivial** change: call `/gsd:fast` inline → ask whether to tweak → if yes call `/gsd:fast` again → once satisfied, squash all commits into 1 and leave one STATE.md row.

**Neighbors (escalate, don't cross):**
- **gsx-fast** (this): any **trivial** change (FE or BE), via `/gsd:fast` **inline**, multi-round + final squash.
- **gsx-quick**: weighted single problem (research/planning/cross-file/API-semantics), via `/gsd:quick --discuss`, leaves PLAN/SUMMARY, **discusses + researches + verifies the change** instead of applying it blind.

**Boundary:** Only orchestrate. All code changes go through `/gsd:fast` (no Edit/Write of source here). Direct non-source ops allowed: ① final squash (`git reset --soft` + re-commit) or undo (`git reset --hard`); ② normalize the STATE.md table to 1 row at close-out; ③ park/restore unrelated uncommitted changes via `git stash` (step 1.6). One trivial change per run; different problem → restart. No branch switching. Reply in Chinese.

**Parked-changes rule:** if step 1.6 ran `git stash push` (recorded `STASH=yes`), the **final action on every exit path** (squash close-out, undo, no-change abort) MUST be `git stash pop` to restore the user's unrelated changes. The no-overlap precondition guarantees a clean pop; on the off chance of a conflict, **stop and report the stash ref** (`git stash list`) for manual recovery — never `git clean`/`rm`.

---

## 0 — Get the problem description

`$ARGUMENTS` = the small change. If empty, ask: *"What small change should be fast-edited? (trivial one-liner)"*. **Record this sentence verbatim in the conversation** — it's the default squash subject (`/gsd:fast` produces no SUMMARY).

## 1 — Baseline + dirty-tree snapshot

```bash
BASELINE=$(git rev-parse HEAD)
git status --short
```
**Write in the conversation** (Skill has no variable persistence): the baseline literal (e.g. *"baseline: 5957369"*) — squash/undo returns here — **and** the list of uncommitted file paths (`DIRTY_FILES`). The dirty-tree **decision is deferred to step 1.6**, because it depends on which files this change will touch (识别于 step 1.5).

## 1.5 — Trivial scope check + escalation (before first /gsd:fast)

`/gsd:fast` is only for **trivial** work — one sentence, ≤3 file edits, ≤1 min, no research/planning/new deps. Read-only assess (Read/Grep/Glob — no guessing). Three exits:

- **Not trivial** (>3 edits, cross-file, needs research/planning/new deps/new components/new routes/architecture) → **escalate to gsx-quick** (or `/gsd:quick`). Tell user: *"Exceeds the fast tier — better for gsx-quick (/gsd:quick --discuss: 讨论 + 研究 + 计划校验 + 验证，留 PLAN/SUMMARY). Switch?"*
- **Value-semantics trap** (looks trivial, belongs in backend) → **escalate to gsx-quick**. Anything changing "what the API returns, or how a return value renders as a label": field localization / status in Chinese / enum·code → readable label / amount·date formatting / masking — correct fix is usually backend (VO/Service/DTO/dict), not a hardcoded Vue map. **Exception:** if read-only confirmed this project does the translation purely on the frontend (frontend dict/i18n/label-map), it's pure-frontend trivial and may stay. **When in doubt → escalate.**
- **Truly trivial** (typo/copy fix, config value, version number, one import / `.gitignore` line, a constant, frontend spacing·color·border-radius, or an already-frontend-owned label translation) → step 2. FE or BE both fine if genuinely trivial and not the two cases above.

Nothing here confirms a change with the user before it lands (that's what gsx-quick's `--discuss` discussion round is for) — escalate anything dangerous/weighted.

**While assessing, also record the file(s) this change will edit (`TARGET_FILES`) in the conversation** — step 1.6 needs them to decide whether to prompt. For a trivial change the read-only pass usually pins this to 1–3 concrete paths; if you can't confidently name them, treat it as *uncertain* in step 1.6.

## 1.6 — Dirty-tree decision (overlap-aware)

`/gsd:fast` uses **`git add -A`** — it sweeps **every** uncommitted change into its commit. Decide by whether this change's files collide with the pre-existing uncommitted ones:

- **Clean tree** (`DIRTY_FILES` empty) → nothing to sweep; go to step 2.
- **Dirty, no overlap** (`TARGET_FILES ∩ DIRTY_FILES = ∅` **and** target files confidently known) → **don't ask**. Park the unrelated changes so the run commits only this change's files:
  ```bash
  git stash push -u -m "gsx-fast-preserve-${BASELINE}"
  ```
  **Record `STASH=yes` in the conversation** and restore via `git stash pop` at every exit (see the Parked-changes rule). Go to step 2.
- **Dirty with overlap, or target files uncertain** → can't cleanly isolate this change. `AskUserQuestion` (header `Dirty tree`): *"待修改文件与未提交变更重叠（或无法确定改动范围）—— /gsd:fast 会用 git add -A 一并提交。先 stash/commit 再来？"* options:
  1. `我先自己处理（stash/commit），稍后再来` → stop; restart later.
  2. `继续（已有改动会被一并提交）` → go to step 2.

## 2 — First call: /gsd:fast

```
Skill("gsd:fast", args=problem description)
```
`/gsd:fast` takes over inline: own scope check (may suggest /gsd:quick), edits inline, `git add -A` atomic commit, and (if STATE.md exists) appends a fast line. Don't interfere. On return → step 3.

## 3 — Check commits this round

No task_dir/slug to identify — check commits:
```bash
git log "${BASELINE}..HEAD" --oneline
```
- **0 commits** → likely judged not trivial / suggested /gsd:quick, or nothing to change. **If `STASH=yes`, `git stash pop` first** (restore the user's parked changes). Relay: *"/gsd:fast produced no commit (likely exceeds trivial scope). HEAD unchanged. Switch to gsx-quick?"*, then stop (step 8 no-change exit).
- **≥1** → show commit list (hash + subject), go to step 4.

## 4 — Interactive prompt

`AskUserQuestion`, header `Continue?`: *"Round done (N commits). Satisfied? Squash to close out; to keep tweaking tell me what to change; to discard select undo."* options:
1. `Satisfied — squash and close out` → step 6.
2. `Undo all — git reset --hard to start` → step 7.
- "Other" + free-text feedback → step 5.

## 5 — Continue tweaking: another /gsd:fast round (no resume)

`/gsd:fast` is **stateless inline** — no `resume`. Each follow-up tweak is a new trivial task stacking a commit on HEAD.

1. **Restate feedback**: *"User feedback: {verbatim}. Calling /gsd:fast again for this tweak."*
2. **Re-run step 1.5 trivial check** on the follow-up — if it pushes over the line (research/cross-file/value-semantics) → prompt escalation to gsx-quick.
3. **Call again**: `Skill("gsd:fast", args=this round's tweak)`
4. **Return to step 3** — re-check `git log BASELINE..HEAD`, ask "Continue?" again.

No hard cap; after every 3 rounds without convergence, prompt: *"3 rounds — multiple rounds often mean it's not trivial. Continue / squash current state / undo all / switch to gsx-quick?"*

## 6 — Squash close-out (satisfied path)

### 6a — Commit subject (no SUMMARY)
1. step 0 task description (`$ARGUMENTS`), normalized ≤80 chars.
2. if too vague → first fast commit's subject (`git log BASELINE..HEAD --oneline | tail -1`).

`type`: infer — `fix` (default), `style` (visual/CSS), `chore` (config/version/deps/build), `docs`, `refactor` (rename/reorg).

### 6b — Final confirmation (ask once)
`AskUserQuestion`, header `Squash confirmation`: *"Squash N commits into 1, subject: `{type}: {subject}`. OK?"* options: 1. `OK, start squash` → 6c.  2. `Let me change the subject` (Other = new subject) → use new subject and **execute squash immediately without re-asking**.

### 6c — Execute squash + normalize STATE.md
```bash
git reset --soft "${BASELINE}"

# Restore STATE.md to baseline: clears stale per-round fast rows (6d adds 1 clean row); || true if no STATE.md at baseline.
git checkout "${BASELINE}" -- .planning/STATE.md 2>/dev/null || true

git add -A
git commit -m "{type}: {subject}

Squashed N fast-edits (gsx-fast):
- {commit1 hash} {commit1 subject}
- {commit2 hash} {commit2 subject}
- ..."
NEW_COMMIT=$(git rev-parse --short HEAD)
```

### 6d — Append 1 row to STATE.md (if the table exists)
Only when `.planning/STATE.md` has the "Quick Tasks Completed" table (5 cols `| # | Description | Date | Commit | Directory |`). **Read** it, find the last data row, **Edit** to insert after it (match that row's full text → replace with "row + newline + new row" for uniqueness):
```
| fast | {one-line: subject or clearer summary} | {YYYY-MM-DD} | {NEW_COMMIT} | — |
```
- Date via `date +%Y-%m-%d`.
- `#` col = literal `fast`; `Directory` col = `—`.
- If a "Last activity:" line follows the table, update its date to today.
- Commit this row separately (keeps the code-commit hash clean):
  ```bash
  git add .planning/STATE.md
  git commit -m "docs(state): log gsx-fast {subject short summary}"
  ```
If STATE.md / the table is absent → **silently skip**.

**Restore parked changes (if `STASH=yes`):** as the final action, `git stash pop` to bring back the user's unrelated uncommitted changes (no-overlap → conflict-free; on conflict, stop and report the stash ref). Go to step 8.

## 7 — Undo all (undo path)

### 7a — Second confirmation
List `BASELINE..HEAD` commits, then `AskUserQuestion` header `Confirm undo`: *"git reset --hard to {BASELINE short}, discarding N commits. Irreversible. Confirm?"* options: 1. `Confirm undo` → 7b.  2. `Never mind, keep tweaking` → step 4.

### 7b — Execute undo
```bash
git reset --hard "${BASELINE}"
```
Reverts all tracked changes after baseline (incl. STATE.md rows). `/gsd:fast` uses `git add -A` so it normally leaves no untracked files, but run `git status --short` anyway; if untracked remain → list them, **let the user decide** — this skill never runs `git clean`/`rm`.

**Restore parked changes (if `STASH=yes`):** after the reset, `git stash pop` to bring back the user's unrelated uncommitted changes (conflict-free under no-overlap; on conflict stop + report the stash ref). Go to step 8.

## 8 — Final summary

**Satisfied (squash):**
```
## gsx-fast — Done
Change:       {problem}
Iterations:   first call + {N-1} tweaks
Commits:      {N} → squashed into 1
Final commit: {NEW_COMMIT} {type}: {subject} (git show {NEW_COMMIT})
STATE.md:     added 1 fast row (if table exists)
```

**Undo:**
```
## gsx-fast — Undone
Change:       {problem}
Discarded {N} commits; HEAD restored to start: {BASELINE short}
```

**No-change exit:**
```
## gsx-fast — Aborted (no changes)
Change:       {problem}
/gsd:fast produced no commit (likely exceeds trivial scope); HEAD unchanged: {BASELINE short}
(Can switch to gsx-quick if needed.)
```
